---
name: starter-nextjs-fullstack
description: Architecture Next.js fullstack (vertical slices) avec Prisma, Better-Auth, Server Actions, TanStack Query, Zod, Jotai, Nuqs, Socket.IO. Déclenché pour tout projet Next.js fullstack avec database et features/.
user-invocable: true
---

# Next.js Fullstack — Architecture Unifiée

Ce skill définit l'architecture d'un projet **Next.js fullstack** avec base de données intégrée.
Il partage le même socle que les starters frontend et mobile (mêmes couches, mêmes conventions).

## Socle commun (partagé avec frontend et mobile)

Voir [shared-architecture.md](shared-architecture.md) pour :
- La structure du dossier `features/` (couches obligatoires)
- Les types globaux (`PaginatedResponse<T>`, `ActionResponse<T>`)
- Le pattern de query key factory + invalidation
- Le module WebSocket (`features/websocket/`)
- Les conventions de nommage
- La gestion d'état (TanStack Query, Jotai, Nuqs)

## Stack spécifique fullstack

| Couche | Outil | Rôle |
|--------|-------|------|
| Framework | Next.js 16+ (App Router) | SSR, routing, Server Actions |
| Auth | Better-Auth (cookies, OAuth) | Session, Google OAuth |
| Database | Prisma 7 + PostgreSQL | ORM, migrations |
| Server data | Server Actions | Couche données primaire |
| HTTP (client) | ak-api-http ou fetch | Client Components (optionnel) |
| Validation | Zod (partagé client/serveur) | Schémas unifiés |
| URL state | Nuqs | Filtres, pagination, tri |
| Client state | Jotai | Modales, config |
| Server state | TanStack React Query | Cache client, invalidation |
| i18n | next-intl | en, fr, ar (RTL) |
| UI | HeroUI + shadcn/ui | Composants |
| Styling | Tailwind CSS v4 (oklch) | Design tokens |
| Real-time | Socket.IO Client | WebSocket |
| Forms | react-hook-form + Zod | Validation |
| Env | @t3-oss/env-nextjs | Validation des variables |
| Quality | Husky + lint-staged | Git hooks |

## Structure du projet

```
app/
  api/
    auth/[...all]/route.ts    # Better-Auth catch-all
    {ressource}/route.ts      # REST API (webhooks, uploads)
  [locale]/
    (protected)/              # Routes protégées
    (public)/                 # Routes publiques

features/
  {feature-name}/
    apis/                     # Fonctions HTTP (client-side, optionnel)
    queries/                  # TanStack Query hooks
    schemas/                  # Schémas Zod (partagés client/serveur)
    types/                    # Interfaces TypeScript + enums
    actions/                  # ⭐ Server Actions (couche données primaire)
    hooks/                    # Hooks spécifiques
    filters/                  # Parsers Nuqs
    utils/                    # Helpers
    components/               # Composants UI
  websocket/                  # Module temps réel

prisma/
  schema.prisma               # Schéma base de données
  migrations/                 # Migrations Prisma

lib/
  auth.ts                     # Config Better-Auth (serveur)
  auth-client.ts              # Better-Auth (client)
  prisma.ts                   # Prisma client singleton
  api-error.ts                # Classe ApiError
  get-query-client.ts         # QueryClient factory
  get-strict-context.tsx      # Context type-safe
  utils.ts                    # cn() et helpers

providers/                    # Auth, Query, Theme, Direction, Jotai
stores/                       # Jotai atoms (modal.store.ts)
hooks/                        # Hooks globaux
components/
  ui/                         # shadcn/ui
  primitives/                 # Layout
  common/                     # Navbar, Footer
config/
  api.ts                      # URLs
  env.ts                      # Validation @t3-oss/env-nextjs
i18n/                         # next-intl
types/                        # Types globaux
socket.ts                     # Factory Socket.IO
```

## Créer un nouveau feature (fullstack)

### 0. Schéma Prisma (`prisma/schema.prisma`)
```prisma
model {Name} {
  id        String   @id @default(cuid())
  // ... champs métier
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  userId    String
  user      User     @relation(fields: [userId], references: [id])

  @@map("{table_name}")
}
```
Puis : `bun run db:generate && bun run db:push`

### 1. Types (`features/{name}/types/{name}.type.ts`)
```typescript
import type { {Name} } from "@/generated/prisma";
export type { {Name} };

export interface I{Name}Params {
  page?: number;
  limit?: number;
  search?: string;
  sortBy?: string;
  sortOrder?: "asc" | "desc";
}
```

### 2. Schemas (`features/{name}/schemas/{name}.schema.ts`)
```typescript
import { z } from "zod";

// Partagé entre react-hook-form (client) et Server Actions (serveur)
export const create{Name}Schema = z.object({
  nom: z.string().min(2, "Le nom doit contenir au moins 2 caractères"),
});

export const update{Name}Schema = create{Name}Schema.partial();
export type Create{Name}DTO = z.infer<typeof create{Name}Schema>;
export type Update{Name}DTO = z.infer<typeof update{Name}Schema>;
```

### 3. Server Actions (`features/{name}/actions/{name}.actions.ts`)
```typescript
"use server";

import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { headers } from "next/headers";
import { revalidatePath } from "next/cache";
import { create{Name}Schema, update{Name}Schema } from "../schemas/{name}.schema";
import type { I{Name}Params } from "../types/{name}.type";

export async function obtenirTous{Name}s(params?: I{Name}Params) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) throw new Error("Non autorisé");

  const { page = 1, limit = 10, search, sortBy = "createdAt", sortOrder = "desc" } = params ?? {};
  const where = search ? { OR: [{ nom: { contains: search, mode: "insensitive" as const } }] } : {};

  const [data, total] = await Promise.all([
    prisma.{name}.findMany({ where, orderBy: { [sortBy]: sortOrder }, skip: (page - 1) * limit, take: limit }),
    prisma.{name}.count({ where }),
  ]);

  return { data, meta: { total, page, limit, totalPages: Math.ceil(total / limit) } };
}

export async function ajouter{Name}(input: unknown) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) throw new Error("Non autorisé");

  const validated = create{Name}Schema.parse(input);
  const result = await prisma.{name}.create({ data: { ...validated, userId: session.user.id } });

  revalidatePath("/{name}s");
  return result;
}

export async function modifier{Name}(id: string, input: unknown) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) throw new Error("Non autorisé");

  const validated = update{Name}Schema.parse(input);
  const result = await prisma.{name}.update({ where: { id }, data: validated });

  revalidatePath("/{name}s");
  return result;
}

export async function supprimer{Name}(id: string) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) throw new Error("Non autorisé");

  await prisma.{name}.delete({ where: { id } });
  revalidatePath("/{name}s");
}
```

### 4. APIs (optionnel, si appel client-side) (`features/{name}/apis/{name}.api.ts`)

Même pattern que le starter frontend. Utilisé seulement si le feature nécessite des appels côté client en plus des Server Actions.

### 5. Query Key Factory (`features/{name}/queries/index.query.ts`)

Identique au starter frontend. Voir [shared-architecture.md](shared-architecture.md).

### 6. Queries (`features/{name}/queries/{name}-list.query.tsx`)
```typescript
import { useQuery } from "@tanstack/react-query";
import { obtenirTous{Name}s } from "../actions/{name}.actions";
import { {name}KeyQuery } from "./index.query";
import type { I{Name}Params } from "../types/{name}.type";

export const use{Name}ListQuery = (params?: I{Name}Params) =>
  useQuery({
    queryKey: {name}KeyQuery("list", params),
    queryFn: () => obtenirTous{Name}s(params),
    staleTime: 30 * 1000,
  });
```

### 7. Mutations (`features/{name}/queries/{name}-add.mutation.tsx`)
```typescript
import { useMutation } from "@tanstack/react-query";
import { addToast } from "@heroui/react";
import { ajouter{Name} } from "../actions/{name}.actions";
import { useInvalidate{Name}Query } from "./index.query";

export const useAjouter{Name}Mutation = () => {
  const invalidate = useInvalidate{Name}Query();
  return useMutation({
    mutationFn: ajouter{Name},
    onSuccess: () => {
      invalidate();
      addToast({ title: "Succès", description: "Élément ajouté", color: "success" });
    },
    onError: (error) => {
      addToast({ title: "Erreur", description: error.message, color: "danger" });
    },
  });
};
```

### 8. Filters (`features/{name}/filters/{name}.filters.ts`)

Identique au starter frontend. Même pattern Nuqs.

## Auth (Better-Auth)

```typescript
// lib/auth.ts — Serveur
export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: "postgresql" }),
  socialProviders: { google: { clientId, clientSecret } },
  plugins: [nextCookies()],
});

// lib/auth-client.ts — Client
export const authClient = createAuthClient({});

// app/api/auth/[...all]/route.ts
export const { POST, GET } = toNextJsHandler(auth);
```

## Différences clés vs Frontend

| Aspect | Frontend | Fullstack |
|--------|----------|-----------|
| Auth | NextAuth 5 (JWT) | Better-Auth (cookies, OAuth) |
| Data layer | `apis/` (ak-api-http) | `actions/` (Server Actions) + `apis/` optionnel |
| Database | Aucune (backend externe) | Prisma 7 + PostgreSQL |
| API routes | Mock seulement | Webhooks, uploads, external integrations |
| Validation | ak-zod-form-kit | Zod direct (partagé client/serveur) |
| Refresh | revalidatePath non utilisé | revalidatePath après chaque mutation |

## Conventions fullstack

- **Server Actions** = couche données primaire (pas les API routes)
- API routes uniquement pour : webhooks, intégrations externes, uploads
- Zod partagé entre `react-hook-form` (client) et Server Actions (serveur)
- Toujours valider dans les Server Actions (ne jamais faire confiance au client)
- `revalidatePath()` après chaque mutation pour rafraîchir les Server Components
- Auth check dans chaque Server Action et API route
