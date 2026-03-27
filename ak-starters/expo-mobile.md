---
name: starter-expo-react-native
description: Architecture Expo React Native (vertical slices) avec ak-api-http, TanStack Query, Zod, Jotai, Socket.IO, Uniwind, HeroUI Native, expo-secure-store. Déclenché pour tout projet mobile Expo avec features/.
user-invocable: true
---

# Expo React Native — Architecture Unifiée

Ce skill définit l'architecture d'un projet **Expo React Native** connecté à un backend externe.
Il partage le même socle que les starters Next.js (mêmes couches, mêmes conventions).

## Socle commun (partagé avec frontend et fullstack)

Voir [shared-architecture.md](shared-architecture.md) pour :
- La structure du dossier `features/` (couches obligatoires)
- Les types globaux (`PaginatedResponse<T>`, `ActionResponse<T>`)
- Le pattern de query key factory + invalidation
- Le module WebSocket (`features/websocket/`)
- Les conventions de nommage
- La gestion d'état

## Stack spécifique mobile

| Couche | Outil | Rôle |
|--------|-------|------|
| Framework | Expo 54+ / React Native 0.81+ | App mobile (iOS, Android, Web) |
| Routing | Expo Router (file-based) | Navigation |
| Auth | AuthContext + expo-secure-store | OTP, JWT, refresh tokens |
| HTTP | ak-api-http (`lib/api.ts`) | Client API unique |
| Validation | Zod | Schémas |
| Client state | Jotai | Modales, config, UI |
| Persistent state | Jotai + AsyncStorage | Notifications, panier, fidélité |
| Server state | TanStack React Query | Cache, invalidation, mutations |
| Real-time | Socket.IO Client | WebSocket |
| Styling | Uniwind + Tailwind CSS v4 | Classes utilitaires via `className` |
| UI | HeroUI Native | Composants natifs (Button, Card, TextField, Dialog) |
| Variants | Tailwind Variants (`tv()`) | Composition de variantes de composants |
| Push | OneSignal + expo-notifications | Notifications |
| Maps | react-native-maps + expo-location | Géolocalisation |
| Storage | expo-secure-store (tokens), AsyncStorage (data) | Persistance |
| Forms | react-hook-form + Zod | Validation |

## Structure du projet

```
src/
  app/                            # Expo Router (file-based)
    _layout.tsx                   # Root layout (providers, fonts, init)
    index.tsx                     # Entry redirect
    (auth)/                       # Auth (login, OTP, register)
    (protected)/                  # Routes protégées
    (public)/                     # Routes publiques
    (tabs)/                       # Navigation par onglets

  features/
    {feature-name}/
      apis/                       # ⭐ Fonctions HTTP (ak-api-http) — PAS "services/"
      queries/                    # TanStack Query (useQuery, useMutation)
      schemas/                    # Schémas Zod
      types/                      # Interfaces TypeScript + enums
      hooks/                      # Hooks spécifiques (dont WebSocket sync)
      utils/                      # Helpers
      components/                 # Composants React Native du feature
    websocket/                    # Module temps réel (Socket.IO)
      types/
      hooks/

  lib/
    api.ts                        # ak-api-http (auth via expo-secure-store)
    api-error.ts                  # Classe ApiError
    get-query-client.ts           # QueryClient + NetInfo + AppState
    get-strict-context.tsx        # Context type-safe
    utils.ts                      # cn() et helpers

  providers/                      # Auth, Query, Theme, Jotai
  stores/                         # Jotai atoms (modal.store.ts, config)
  hooks/                          # Hooks globaux (useRealtimeUpdates, useResponsive)
  components/
    ui/                           # Primitives UI
    common/                       # Composants partagés
  config/
    environment.ts                # Variables d'environnement (app.config.js)
    api.ts                        # baseURL, baseFileURL
  types/                          # PaginatedResponse<T>, ActionResponse<T>
  constants/                      # Constantes globales
  utils/                          # Utilitaires globaux
  assets/                         # Images, fonts, sons
  socket.ts                       # Factory Socket.IO
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

### 3. APIs (`features/{name}/apis/{name}.api.ts`) — PAS "services/"
```typescript
import { api } from "@/lib/api";
import type { PaginatedResponse } from "@/types/api.type";
import type { I{Name}, I{Name}Params, Create{Name}DTO, Update{Name}DTO } from "../types/{name}.type";

const BASE = "/{ressource}";

export const {name}API = {
  obtenirTous: (params?: I{Name}Params): Promise<PaginatedResponse<I{Name}>> =>
    api.get(BASE, { params }),
  obtenirParId: (id: string): Promise<I{Name}> =>
    api.get(`${BASE}/${id}`),
  ajouter: (data: Create{Name}DTO): Promise<I{Name}> =>
    api.post(BASE, data),
  modifier: (id: string, data: Update{Name}DTO): Promise<I{Name}> =>
    api.patch(`${BASE}/${id}`, data),
  supprimer: (id: string): Promise<void> =>
    api.delete(`${BASE}/${id}`),
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
    staleTime: 60 * 1000,
  });
```

### 6. Mutation (`features/{name}/queries/{name}-add.mutation.tsx`)
```typescript
import { useMutation } from "@tanstack/react-query";
import { {name}API } from "../apis/{name}.api";
import { useInvalidate{Name}Query } from "./index.query";

export const useAjouter{Name}Mutation = () => {
  const invalidate = useInvalidate{Name}Query();
  return useMutation({
    mutationFn: (data: Create{Name}DTO) => {name}API.ajouter(data),
    onSuccess: () => invalidate(),
  });
};
```

### 7. WebSocket Sync Hook (`features/{name}/hooks/use{Name}SocketSync.ts`)
```typescript
import { useEffect, useCallback } from "react";
import { useQueryClient } from "@tanstack/react-query";
import { getSocket } from "@/socket";
import { {name}KeyQuery } from "../queries/index.query";

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

### 8. Screen (`app/(protected)/{name}/index.tsx`)
```tsx
import { View, FlatList, ActivityIndicator } from "react-native";
import { Typography } from "@/components/ui/typography";
import { cn } from "heroui-native";
import { use{Name}ListQuery } from "@/features/{name}/queries/{name}-list.query";
import { use{Name}SocketSync } from "@/features/{name}/hooks/use{Name}SocketSync";

export default function {Name}Screen() {
  const { data, isLoading, refetch } = use{Name}ListQuery();
  use{Name}SocketSync();

  if (isLoading) return <ActivityIndicator />;

  return (
    <FlatList
      className="flex-1 bg-background"
      data={data?.data}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View className="p-4 border-b border-separator">
          <Typography variant="body" weight="bold">{item.nom}</Typography>
        </View>
      )}
      onRefresh={refetch}
      refreshing={isLoading}
      contentInsetAdjustmentBehavior="automatic"
    />
  );
}
```

## Client API (ak-api-http)

```typescript
// lib/api.ts
import { Api } from "ak-api-http";
import * as SecureStore from "expo-secure-store";
import { baseURL } from "@/config/api";

export const api = new Api({
  baseUrl: baseURL,
  timeout: 10000, maxRetries: 3, retryDelay: 1000,
  enableAuth: true,
  getSession: async () => {
    const token = await SecureStore.getItemAsync("auth_token");
    return { accessToken: token ?? "" };
  },
  signOut: async () => {
    await SecureStore.deleteItemAsync("auth_token");
  },
  debug: __DEV__,
});
```

## Auth (OTP + JWT + expo-secure-store)

1. Saisie téléphone → `POST /auth/customer/login` → OTP envoyé
2. Vérification OTP → `POST /auth/customer/verify-otp` → JWT retourné
3. Token stocké dans `expo-secure-store`
4. ak-api-http injecte le token via `getSession()`
5. 401 → auto-refresh via `/auth/refresh`
6. Au lancement → session restaurée depuis SecureStore → validation silencieuse

## React Query (mobile)

```typescript
// lib/get-query-client.ts
import { onlineManager, focusManager } from "@tanstack/react-query";
import NetInfo from "@react-native-community/netinfo";
import { AppState } from "react-native";

// Sync avec le réseau
onlineManager.setEventListener((setOnline) =>
  NetInfo.addEventListener((state) => setOnline(!!state.isConnected))
);

// Pause quand l'app est en arrière-plan
focusManager.setEventListener((setFocused) => {
  const sub = AppState.addEventListener("change", (s) => setFocused(s === "active"));
  return () => sub.remove();
});
```

## Différences clés vs Next.js

| Aspect | Next.js Frontend | Expo Mobile |
|--------|-----------------|-------------|
| Auth | NextAuth 5 (JWT) | AuthContext + expo-secure-store |
| HTTP | ak-api-http dual (serveur/client) | ak-api-http unique |
| URL state | Nuqs | Pas de Nuqs (pas d'URL) |
| Persistent state | — | Jotai + AsyncStorage |
| Offline | — | NetInfo + React Query pause |
| Notifications | — | OneSignal + expo-notifications |
| Navigation | App Router | Expo Router |
| Styling | Tailwind CSS v4 | Uniwind + Tailwind CSS v4 |
| UI | HeroUI (web) | HeroUI Native |
| Toast | HeroUI addToast | HeroUI Native toast / Alert |

## Styling (Uniwind + HeroUI Native + Tailwind Variants)

### Setup Metro
```javascript
// metro.config.js
const { withUniwindConfig } = require('uniwind/metro');
module.exports = withUniwindConfig(config, {
  cssEntryFile: './global.css',
  dtsFile: './uniwind-types.d.ts',
});
```

### global.css (design tokens oklch)
```css
@import 'tailwindcss';
@import 'uniwind';
@import 'heroui-native/styles';
@source './node_modules/heroui-native/lib';

@layer theme {
  :root {
    @variant light { /* tokens light */ }
    @variant dark { /* tokens dark */ }
  }
}
```

### Patterns de styling

**1. className directement** (composants RN supportés par Uniwind)
```tsx
<View className="flex-1 bg-background p-4">
  <Text className="text-foreground text-lg font-bold">Titre</Text>
</View>
```

**2. cn() pour merger les classes** (depuis heroui-native)
```tsx
import { cn } from "heroui-native";
<View className={cn("flex-1 bg-background", className)} />
```

**3. withUniwind() pour les composants non supportés**
```tsx
import { withUniwind } from "uniwind";
import { Image } from "expo-image";
const StyledImage = withUniwind(Image);
<StyledImage className="h-32 w-full rounded-lg" />
```

**4. Tailwind Variants (tv) pour les composants réutilisables**
```tsx
import { tv, type VariantProps } from "tailwind-variants";

const buttonVariants = tv({
  base: "rounded-lg px-4 py-3 items-center justify-center",
  variants: {
    variant: {
      primary: "bg-accent",
      secondary: "bg-surface",
      danger: "bg-danger",
    },
    size: {
      sm: "px-3 py-2",
      md: "px-4 py-3",
      lg: "px-6 py-4",
    },
  },
  defaultVariants: { variant: "primary", size: "md" },
});
```

**5. HeroUI Native composants**
```tsx
import { Button, TextField, Card } from "heroui-native";
<Button variant="ghost" className="rounded-full w-12 h-12" onPress={handler} />
```

**6. Thème (light/dark)**
```tsx
import { Uniwind, useUniwind } from "uniwind";
import { useThemeColor } from "heroui-native";

const { theme } = useUniwind();
Uniwind.setTheme(theme === "light" ? "dark" : "light");

const accentColor = useThemeColor("accent"); // valeur calculée
```

## Conventions mobile

- `apis/` et JAMAIS `services/` — même nom que les starters web
- ak-api-http est le client HTTP unique (pas Axios en parallèle)
- Jotai pour l'état client (pas Zustand) — cohérence avec les starters web
- expo-secure-store pour les tokens (pas AsyncStorage)
- AsyncStorage uniquement pour les données non sensibles via Jotai persist
- `useRealtimeUpdates()` dans le layout protégé comme hub central WebSocket
- Pull-to-refresh sur tous les écrans de liste
- `contentInsetAdjustmentBehavior="automatic"` sur ScrollView/FlatList
- Kebab-case pour les noms de fichiers de composants
- Routes dans `app/`, logique métier dans `features/`
- **Uniwind** pour le styling (PAS NativeWind) — `className` + `cn()` + `tv()`
- **HeroUI Native** pour les composants UI (Button, Card, TextField, Dialog)
- Design tokens en oklch dans `global.css` avec `@variant light/dark`
- `withUniwind()` HOC uniquement pour les composants tiers non supportés
