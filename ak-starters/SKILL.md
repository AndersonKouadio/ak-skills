---
name: ak-starters
description: Architecture unifiée vertical slices pour les projets ak-*. Guide le scaffolding de features (apis, queries, schemas, types, hooks) pour Next.js frontend, Next.js fullstack et Expo React Native. Déclenché quand on travaille sur un projet avec features/, ak-api-http, ou quand on crée un nouveau feature/module.
---

# ak-starters — Architecture Unifiée

Ce skill guide la création et le développement de projets suivant l'architecture **vertical slices** de l'écosystème ak-*.

Trois domaines, un seul socle.

## Architecture de base

Lire **[shared-architecture.md](shared-architecture.md)** en premier. C'est le contrat commun :
- 7 couches obligatoires dans `features/` (`apis/`, `queries/`, `schemas/`, `types/`, `hooks/`, `utils/`, `components/`)
- Types globaux (`PaginatedResponse<T>`, `ActionResponse<T>`)
- Pattern API object, Query Key Factory, Schemas Zod
- Module WebSocket (`features/websocket/`)
- Gestion d'état (TanStack Query + Jotai + Nuqs)
- Conventions de nommage (français)
- Modal Store partagé
- Client HTTP ak-api-http
- Checklist nouveau feature

## Choisir le bon domaine

| Si le projet a... | Domaine | Fichier de référence |
|--------------------|---------|---------------------|
| `next.config` + backend externe | Next.js Frontend | [next-frontend.md](next-frontend.md) |
| `next.config` + `prisma/` | Next.js Fullstack | [next-fullstack.md](next-fullstack.md) |
| `app.config.js` + Expo | Expo Mobile | [expo-mobile.md](expo-mobile.md) |

## Résumé des différences

| Aspect | Frontend | Fullstack | Mobile |
|--------|----------|-----------|--------|
| Auth | NextAuth (JWT) | Better-Auth (cookies) | expo-secure-store (OTP) |
| Data layer | `apis/` (ak-api-http) | `actions/` + `apis/` | `apis/` (ak-api-http) |
| Database | Aucune | Prisma + PostgreSQL | Aucune |
| URL state | Nuqs | Nuqs | — |
| Styling | Tailwind CSS v4 | Tailwind CSS v4 | Uniwind + Tailwind CSS v4 |
| UI lib | HeroUI + shadcn/ui | HeroUI + shadcn/ui | HeroUI Native |
| Offline | — | — | NetInfo + React Query |

## Utilisation

Quand on te demande de **créer un feature**, **ajouter un module**, ou **scaffolder** :

1. Identifie le domaine (frontend / fullstack / mobile)
2. Lis `shared-architecture.md` pour les couches et conventions
3. Lis le fichier domaine correspondant pour les spécificités
4. Génère TOUTES les couches (pas seulement une partie)
5. Respecte la checklist de `shared-architecture.md`
