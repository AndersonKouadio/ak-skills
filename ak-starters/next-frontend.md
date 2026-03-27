---
name: starter-nextjs-frontend
description: Architecture Next.js frontend (vertical slices) connecté à un backend externe. Utilise ak-api-http, TanStack Query, Zod, Jotai, Nuqs, NextAuth, Socket.IO, HeroUI, next-intl. Déclenché pour tout projet Next.js frontend avec features/.
user-invocable: true
---

# Next.js Frontend — Architecture Unifiée

Ce skill définit l'architecture d'un projet **Next.js frontend** connecté à un backend externe.
Il partage le même socle que les starters fullstack et mobile (mêmes couches, mêmes conventions).

## Socle commun (partagé avec fullstack et mobile)

Voir [shared-architecture.md](shared-architecture.md) pour :
- La structure du dossier `features/` (couches obligatoires)
- Les types globaux (`PaginatedResponse<T>`, `ActionResponse<T>`)
- Le pattern de query key factory + invalidation
- Le module WebSocket (`features/websocket/`)
- Les conventions de nommage
- La gestion d'état (TanStack Query, Jotai, Nuqs)

## Stack spécifique frontend

| Couche | Outil | Rôle |
|--------|-------|------|
| Framework | Next.js 16+ (App Router) | SSR, routing, middleware |
| Auth | NextAuth 5 (JWT + refresh) | Session, tokens, guards |
| HTTP (serveur) | ak-api-http (`lib/api.ts`) | RSC, Server Actions |
| HTTP (client) | ak-api-http (`lib/api.client.ts`) | Client Components |
| Validation forms | ak-zod-form-kit | `processAndValidateFormData()` |
| URL state | Nuqs | Filtres, pagination, tri |
| Client state | Jotai | Modales, config, sidebar |
| Server state | TanStack React Query | Cache, invalidation, mutations |
| i18n | next-intl | en, fr, ar (RTL) |
| UI | HeroUI + shadcn/ui | Composants |
| Styling | Tailwind CSS v4 (oklch) | Design tokens |
| Real-time | Socket.IO Client | WebSocket |
| Forms | react-hook-form + Zod | Validation |
| Env | @t3-oss/env-nextjs | Validation des variables |
| Quality | Husky + lint-staged | Git hooks |

## Structure du projet

```
app/[locale]/
  (protected)/                # Routes protégées (middleware vérifie JWT)
  (public)/                   # Routes publiques
  api/                        # Mock API (dev sans backend)

features/
  {feature-name}/
    apis/                     # Fonctions HTTP (ak-api-http)
    queries/                  # TanStack Query (useQuery, useMutation)
    schemas/                  # Schémas Zod
    types/                    # Interfaces TypeScript + enums
    hooks/                    # Hooks spécifiques au feature
    filters/                  # Parsers Nuqs (URL state)
    utils/                    # Helpers
    components/               # Composants UI du feature
  websocket/                  # Module temps réel (Socket.IO)
    types/
    hooks/

lib/
  api.ts                      # ak-api-http (serveur — auth())
  api.client.ts               # ak-api-http (client — getSession())
  api-error.ts                # Classe ApiError typée
  auth.ts                     # Config NextAuth (JWT, refresh tokens)
  get-query-client.ts         # QueryClient factory
  get-strict-context.tsx      # Utilitaire React Context type-safe
  utils.ts                    # cn() et helpers

providers/                    # Auth, Query, Theme, Direction, Mounted, Jotai
stores/                       # Jotai atoms (modal.store.ts, config)
hooks/                        # Hooks globaux (useConfig, useMediaQuery)
components/
  ui/                         # shadcn/ui
  primitives/                 # Layout (Title, Section, Display, Main)
  common/                     # Navbar, Footer
config/
  api.ts                      # baseURL, baseFileURL
  env.ts                      # Validation @t3-oss/env-nextjs
  site.ts                     # Metadata
  fonts.ts                    # Polices
i18n/                         # next-intl (messages par locale)
types/                        # PaginatedResponse<T>, ActionResponse<T>, next-auth.d.ts
socket.ts                     # Factory Socket.IO (racine)
```

## Créer un nouveau feature

### 1. Types (`features/{name}/types/{name}.type.ts`)
```typescript
export enum {Name}Statut {
  ACTIF = "ACTIF",
  INACTIF = "INACTIF",
}

export interface I{Name} {
  id: string;
  // ... champs métier
  createdAt: string;
  updatedAt: string;
}

export interface I{Name}Params {
  page?: number;
  limit?: number;
  search?: string;
  statut?: {Name}Statut;
  sortBy?: string;
  sortOrder?: "asc" | "desc";
}
```

### 2. Schemas (`features/{name}/schemas/{name}.schema.ts`)
```typescript
import { z } from "zod";

export const create{Name}Schema = z.object({
  nom: z.string().min(2, "Le nom doit contenir au moins 2 caractères"),
});

export const update{Name}Schema = create{Name}Schema.partial();

export type Create{Name}DTO = z.infer<typeof create{Name}Schema>;
export type Update{Name}DTO = z.infer<typeof update{Name}Schema>;
```

### 3. APIs (`features/{name}/apis/{name}.api.ts`)
```typescript
import { apiClient } from "@/lib/api.client";
import type { PaginatedResponse } from "@/types/api.type";
import type { I{Name}, I{Name}Params, Create{Name}DTO, Update{Name}DTO } from "../types/{name}.type";

const BASE = "/{ressource}";

export const {name}API = {
  obtenirTous: (params?: I{Name}Params): Promise<PaginatedResponse<I{Name}>> =>
    apiClient.get(BASE, { params }),
  obtenirParId: (id: string): Promise<I{Name}> =>
    apiClient.get(`${BASE}/${id}`),
  ajouter: (data: Create{Name}DTO): Promise<I{Name}> =>
    apiClient.post(BASE, data),
  modifier: (id: string, data: Update{Name}DTO): Promise<I{Name}> =>
    apiClient.patch(`${BASE}/${id}`, data),
  supprimer: (id: string): Promise<void> =>
    apiClient.delete(`${BASE}/${id}`),
};
```

### 4. Query Key Factory (`features/{name}/queries/index.query.ts`)
```typescript
import { useQueryClient } from "@tanstack/react-query";

export const {name}KeyQuery = (...params: unknown[]) => ["{name}", ...params];

export const useInvalidate{Name}Query = () => {
  const queryClient = useQueryClient();
  return () => queryClient.invalidateQueries({ queryKey: ["{name}"] });
};
```

### 5. List Query (`features/{name}/queries/{name}-list.query.tsx`)
```typescript
import { useQuery } from "@tanstack/react-query";
import { {name}API } from "../apis/{name}.api";
import { {name}KeyQuery } from "./index.query";
import type { I{Name}Params } from "../types/{name}.type";

export const use{Name}ListQuery = (params?: I{Name}Params) =>
  useQuery({
    queryKey: {name}KeyQuery("list", params),
    queryFn: () => {name}API.obtenirTous(params),
    staleTime: 30 * 1000,
  });
```

### 6. Mutation (`features/{name}/queries/{name}-add.mutation.tsx`)
```typescript
import { useMutation } from "@tanstack/react-query";
import { addToast } from "@heroui/react";
import { processAndValidateFormData } from "ak-zod-form-kit";
import { create{Name}Schema } from "../schemas/{name}.schema";
import { {name}API } from "../apis/{name}.api";
import { useInvalidate{Name}Query } from "./index.query";

export const useAjouter{Name}Mutation = () => {
  const invalidate = useInvalidate{Name}Query();
  return useMutation({
    mutationFn: async ({ data }: { data: FormData | Record<string, unknown> }) => {
      const validation = processAndValidateFormData(create{Name}Schema, data);
      if (!validation.success) throw new Error(validation.errorsInString);
      return {name}API.ajouter(validation.data);
    },
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

### 7. Filters (`features/{name}/filters/{name}.filters.ts`)
```typescript
import { parseAsInteger, parseAsString, parseAsStringEnum } from "nuqs";
import { {Name}Statut } from "../types/{name}.type";

export const {name}FiltersClient = {
  filter: {
    search: parseAsString.withDefault(""),
    statut: parseAsStringEnum<{Name}Statut>(Object.values({Name}Statut)),
    page: parseAsInteger.withDefault(1),
    limit: parseAsInteger.withDefault(10),
  },
  option: { clearOnDefault: true, throttleMs: 500 },
};
```

## Client API dual

```typescript
// lib/api.ts (SERVEUR — RSC, Server Actions, API routes)
import { Api } from "ak-api-http";
import { auth } from "./auth";

export const api = new Api({
  baseUrl: baseURL,
  timeout: 10000, maxRetries: 3, retryDelay: 1000,
  enableAuth: true,
  getSession: async () => {
    const session = await auth();
    return { accessToken: session?.user?.accessToken ?? "" };
  },
  signOut: async () => { await signOut(); },
  debug: true,
});

// lib/api.client.ts (CLIENT — Client Components)
"use client";
import { Api } from "ak-api-http";
import { getSession } from "next-auth/react";

export const apiClient = new Api({
  baseUrl: baseURL,
  timeout: 10000, maxRetries: 3, retryDelay: 1000,
  enableAuth: true,
  getSession: async () => {
    const session = await getSession();
    return { accessToken: (session?.user as any)?.accessToken ?? "" };
  },
  debug: process.env.NODE_ENV === "development",
});
```

## Auth (NextAuth 5 JWT)

- Session strategy : **JWT** (access token + refresh token)
- Refresh automatique quand le token expire
- Session enrichie : id, email, name, role, status, tokens
- Middleware `proxy.ts` protège les routes `(protected)/`
- Routes publiques : `/`, `/auth/*`

## WebSocket (Socket.IO)

Module partagé dans `features/websocket/`. Voir [shared-architecture.md](shared-architecture.md).
- Bootstrap dans le layout protégé via `useWebsocketBootstrap()`
- Invalidation automatique du cache TanStack Query sur événements

## Provider Stack

```
NextIntlClientProvider
  JotaiProvider
    QueryProvider (TanStack Query + DevTools)
      ThemeProviders (HeroUI + next-themes)
        ToastProvider (HeroUI)
          NuqsAdapter
            AuthProvider (NextAuth SessionProvider)
              DirectionProvider (RTL/LTR)
                {children}
```

## État

| Type | Outil | Usage |
|------|-------|-------|
| Serveur | TanStack Query | API, listes, CRUD |
| URL | Nuqs | Filtres, pagination, tri |
| Client global | Jotai | Modales, thème, sidebar |
| Formulaires | react-hook-form | Inputs, validation |
