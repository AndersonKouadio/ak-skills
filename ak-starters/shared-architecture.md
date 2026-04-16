# Architecture Partagée — Vertical Slices

Ce document définit le **socle commun** entre les trois domaines (Next.js Frontend, Next.js Fullstack, Expo React Native). Toute couche définie ici DOIT être respectée dans chaque domaine. Les domaines peuvent ajouter des couches spécifiques (ex: `actions/` pour fullstack, `stores/` pour mobile) mais ne doivent JAMAIS renommer les couches de base.

## Couches obligatoires d'un feature

Chaque feature dans `features/{name}/` DOIT contenir ces couches :

```
features/{name}/
  apis/           # Fonctions HTTP (JAMAIS "services/")
  queries/        # TanStack Query hooks (useQuery, useMutation)
  schemas/        # Schémas Zod de validation
  types/          # Interfaces TypeScript + enums
  hooks/          # Hooks spécifiques au feature
  utils/          # Helpers spécifiques
  components/     # Composants UI spécifiques
```

### Couches optionnelles (par domaine)

| Couche | Frontend | Fullstack | Mobile | Rôle |
|--------|----------|-----------|--------|------|
| `filters/` | Oui (Nuqs) | Oui (Nuqs) | Non | URL state (filtres, pagination) |
| `actions/` | Non | Oui | Non | Server Actions (Prisma) |

### Limite de taille des fichiers

**Un fichier composant ne doit JAMAIS dépasser 150 lignes.** Si un fichier dépasse cette limite :
- Extraire les sous-composants dans des fichiers séparés dans le même dossier `components/`
- Extraire les hooks/logique dans `hooks/`
- Extraire les types dans `types/`
- Extraire les constantes/mock data dans `utils/` ou `mocks/`

### ERREUR courante

`services/` n'existe PAS. C'est toujours `apis/`. Si un projet existant utilise `services/`, c'est une dette technique à corriger lors du refactoring.

## Conventions de nommage

### Fichiers dans chaque couche

| Couche | Pattern de fichier | Exemple |
|--------|-------------------|---------|
| `types/` | `{name}.type.ts` | `utilisateur.type.ts` |
| `schemas/` | `{name}.schema.ts` | `utilisateur.schema.ts` |
| `apis/` | `{name}.api.ts` | `utilisateur.api.ts` |
| `queries/` | `index.query.ts` | Query key factory + invalidation |
| `queries/` | `{name}-list.query.tsx` | `useUtilisateurListQuery()` |
| `queries/` | `{name}-add.mutation.tsx` | `useAjouterUtilisateurMutation()` |
| `queries/` | `{name}-update.mutation.tsx` | `useModifierUtilisateurMutation()` |
| `queries/` | `{name}-delete.mutation.tsx` | `useSupprimerUtilisateurMutation()` |
| `filters/` | `{name}.filters.ts` | `utilisateur.filters.ts` |
| `hooks/` | `use{Name}*.ts` | `useUtilisateurListTable.ts` |
| `actions/` | `{name}.actions.ts` | `utilisateur.actions.ts` (fullstack) |

### Nommage du code

- Noms de features : **français** (`utilisateur`, `departement`, `audit`, `commande`)
- Méthodes API : **français** (`obtenirTous`, `obtenirParId`, `ajouter`, `modifier`, `supprimer`)
- Noms de mutations : **français** (`useAjouter{Name}Mutation`, `useModifier{Name}Mutation`)
- Messages toast / erreurs Zod : **français**
- Interfaces : préfixe `I` (`IUtilisateur`, `IAudit`)
- Enums : PascalCase (`UtilisateurStatut`, `AuditType`)
- Params : suffixe `Params` (`IUtilisateurParams`)
- DTOs : suffixe `DTO` (`CreateUtilisateurDTO`)

## Types globaux

Ces types DOIVENT exister dans `types/api.type.ts` de chaque projet :

```typescript
export enum SortOrder {
  ASC = "asc",
  DESC = "desc",
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export interface ActionResponse<T> {
  success: boolean;
  data?: T;
  message?: string;
  error?: string;
}
```

## Pattern : API object

Toutes les APIs DOIVENT suivre ce pattern identique (3 domaines) :

```typescript
// features/{name}/apis/{name}.api.ts
import { api } from "@/lib/api"; // ou apiClient selon le contexte
import type { PaginatedResponse } from "@/types/api.type";

const BASE = "/{ressource}";

export const {name}API = {
  obtenirTous: (params?) => api.get(BASE, { params }),
  obtenirParId: (id) => api.get(`${BASE}/${id}`),
  ajouter: (data) => api.post(BASE, data),
  modifier: (id, data) => api.patch(`${BASE}/${id}`, data),
  supprimer: (id) => api.delete(`${BASE}/${id}`),
};
```

## Pattern : Query Key Factory

Chaque feature DOIT avoir son factory dans `queries/index.query.ts` :

```typescript
import { useQueryClient } from "@tanstack/react-query";

export const {name}KeyQuery = (...params: unknown[]) => ["{name}", ...params];

export const useInvalidate{Name}Query = () => {
  const queryClient = useQueryClient();
  return () => queryClient.invalidateQueries({ queryKey: ["{name}"] });
};
```

## Pattern : Schemas Zod

```typescript
// features/{name}/schemas/{name}.schema.ts
import { z } from "zod";

export const create{Name}Schema = z.object({
  nom: z.string().min(2, "Le nom doit contenir au moins 2 caractères"),
  // ... champs avec messages d'erreur en français
});

export const update{Name}Schema = create{Name}Schema.partial();

export type Create{Name}DTO = z.infer<typeof create{Name}Schema>;
export type Update{Name}DTO = z.infer<typeof update{Name}Schema>;
```

## Module WebSocket (`features/websocket/`)

Ce module est **partagé entre les 3 domaines**. Il utilise Socket.IO.

### Structure

```
features/websocket/
  types/realtime.type.ts     # Types d'événements
  hooks/useSocket.ts         # Connexion/déconnexion
  hooks/useRealtimeUpdates.ts  # Hub central (invalidation cache)
  hooks/useWebsocketBootstrap.ts  # Bootstrap (layout protégé)
```

### Convention des événements

Format : `{module}:{action}` — ex: `utilisateur:created`, `commande:updated`, `notification:new`

### Intégration avec TanStack Query

Chaque événement Socket.IO invalide les clés de cache correspondantes :

```typescript
const EVENT_QUERY_MAP: Record<RealtimeEvent, readonly string[]> = {
  "utilisateur:created": ["utilisateur"],
  "utilisateur:updated": ["utilisateur"],
  "utilisateur:deleted": ["utilisateur"],
  "notification:new": ["notification"],
};
```

### Hook de sync par feature

Chaque feature qui a besoin de temps réel ajoute un hook `use{Name}SocketSync.ts` dans ses `hooks/` :

```typescript
// features/{name}/hooks/use{Name}SocketSync.ts
export const use{Name}SocketSync = () => {
  const queryClient = useQueryClient();
  const invalidate = useCallback(
    () => queryClient.invalidateQueries({ queryKey: ["{name}"] }),
    [queryClient]
  );

  useEffect(() => {
    const socket = getSocket();
    if (!socket) return;
    socket.on("{name}:created", invalidate);
    socket.on("{name}:updated", invalidate);
    socket.on("{name}:deleted", invalidate);
    return () => {
      socket.off("{name}:created", invalidate);
      socket.off("{name}:updated", invalidate);
      socket.off("{name}:deleted", invalidate);
    };
  }, [invalidate]);
};
```

## Gestion d'état

| Type | Outil | Usage | Tous domaines |
|------|-------|-------|---------------|
| Serveur | TanStack React Query | API, listes, CRUD, cache | Oui |
| Client global | Jotai | Modales, thème, sidebar, config | Oui |
| URL | Nuqs | Filtres, pagination, tri | Web seulement |
| Formulaires | react-hook-form + Zod | Inputs, validation | Oui |
| Persistant | Jotai + persist | Données qui survivent au redémarrage | Mobile surtout |

### Jotai est le standard pour l'état client

- **Pas Zustand** — Jotai est utilisé dans les 3 domaines pour la cohérence
- Pour le mobile, utiliser `jotai/utils` avec `atomWithStorage` et un adapter AsyncStorage
- Le modal store (`stores/modal.store.ts`) est identique dans tous les domaines

### Modal Store (partagé)

```typescript
// stores/modal.store.ts
import { atom, useAtom } from "jotai";

export type ModuleName = "utilisateur"; // Étendre selon le projet
export type ModalAction = "add" | "edit" | "delete" | "lockUnlock" | "view";

export interface ModalState<T = unknown> {
  isOpen: boolean;
  module: ModuleName | null;
  action: ModalAction | null;
  data: T | null;
}

const modalAtom = atom<ModalState>({ isOpen: false, module: null, action: null, data: null });

export function useModalStore<T = unknown>() {
  const [state, setState] = useAtom(modalAtom);
  return {
    ...state,
    data: state.data as T | null,
    openModal: (module: ModuleName, action: ModalAction, data: T | null = null) =>
      setState({ isOpen: true, module, action, data }),
    closeModal: () => setState((prev) => ({ ...prev, isOpen: false })),
  };
}
```

## Client HTTP (ak-api-http)

| Domaine | Fichier | Auth via |
|---------|---------|----------|
| Frontend (serveur) | `lib/api.ts` | `auth()` (NextAuth server) |
| Frontend (client) | `lib/api.client.ts` | `getSession()` (NextAuth client) |
| Fullstack (serveur) | `lib/api.ts` | `auth.api.getSession()` (Better-Auth) |
| Fullstack (client) | `lib/api.client.ts` | `authClient` (Better-Auth client) |
| Mobile | `lib/api.ts` | `expo-secure-store` (token direct) |

Configuration commune :
```typescript
new Api({
  baseUrl: baseURL,
  timeout: 10000,
  maxRetries: 3,
  retryDelay: 1000,
  enableAuth: true,
  getSession: async () => ({ accessToken: "..." }),
  debug: isDev,
});
```

## ApiError (partagé)

```typescript
// lib/api-error.ts
export class ApiError<T = unknown> extends Error {
  status: number;
  body?: ActionResponse<T>;

  constructor(status: number, message: string, body?: ActionResponse<T>) {
    super(message);
    this.name = "ApiError";
    this.status = status;
    this.body = body;
  }
}
```

## QueryClient (partagé)

Configuration de base identique :
```typescript
new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,       // 1 minute
      gcTime: 5 * 60 * 1000,      // 5 minutes
      retry: (count, error: any) => {
        if (error?.response?.status === 404) return false;
        return count < 3;
      },
    },
  },
});
```

Ajouts mobile : `onlineManager` (NetInfo), `focusManager` (AppState).

## Checklist nouveau feature

- [ ] `types/{name}.type.ts` — Interface, enum, params
- [ ] `schemas/{name}.schema.ts` — Zod create + update + DTOs
- [ ] `apis/{name}.api.ts` — obtenirTous, obtenirParId, ajouter, modifier, supprimer
- [ ] `queries/index.query.ts` — Key factory + invalidation hook
- [ ] `queries/{name}-list.query.tsx` — useQuery
- [ ] `queries/{name}-add.mutation.tsx` — useMutation + toast + invalidation
- [ ] `hooks/use{Name}SocketSync.ts` — Si temps réel nécessaire
- [ ] `filters/{name}.filters.ts` — Si web avec Nuqs
- [ ] `actions/{name}.actions.ts` — Si fullstack avec Prisma
- [ ] `components/` — Composants UI du feature
