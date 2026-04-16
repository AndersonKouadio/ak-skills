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

## Workflow de demande — Règles obligatoires

### Avant de coder : demander les assets manquants

**AVANT de commencer à implémenter un écran/feature, vérifier si tous les assets nécessaires sont disponibles :**

1. **Illustrations/images** — Si la maquette montre des illustrations (personnages, objets, icônes custom), vérifier qu'elles existent dans `assets/`. Si non, **DEMANDER à l'utilisateur** de les fournir avant de coder.
2. **Icônes spécifiques** — Si la maquette utilise des icônes qui ne sont pas dans la bibliothèque d'icônes du projet (SF Symbols, MaterialIcons, etc.), le signaler.
3. **Fonts custom** — Si la maquette utilise une police spécifique non installée, le demander.
4. **Assets de marque** — Logos, patterns, backgrounds spécifiques.

**Ne JAMAIS remplacer un asset manquant par :**
- Des émojis (🛵, 📱, etc.)
- Des Views/formes géométriques bricolées à la main
- Des icônes approximatives

**Poser la question explicitement :**
> "La maquette montre [description de l'élément]. Je n'ai pas trouvé cet asset dans le projet. Peux-tu me fournir l'image ?"

### Reproduire la maquette fidèlement (pixel-perfect)

- **Toujours comparer avec la maquette** avant et après l'implémentation
- **Chaque écran doit être testé visuellement** sur le simulateur
- **Les couleurs, espacements, tailles de police, border-radius** doivent correspondre exactement
- **Ne pas interpréter** la maquette — la reproduire telle quelle
- Si un élément de la maquette n'est pas clair, **demander clarification** plutôt que deviner

### Mapper la maquette aux composants UI AVANT de coder

**Workflow obligatoire pour chaque écran/feature :**

1. **Analyser la maquette** et identifier chaque élément visuel (header, cards, listes, inputs, boutons, séparateurs, tabs, badges, avatars, etc.)
2. **Mapper chaque élément au composant de la librairie UI** (HeroUI Native pour mobile, HeroUI React pour web, shadcn/ui si applicable)
   - Lister les composants : `node scripts/list_components.mjs` ou consulter la doc
   - Chercher la doc du composant : `node scripts/get_component_docs.mjs NomComposant`
3. **Vérifier la customisabilité** — pour chaque composant mappé, confirmer qu'il peut être stylisé pour matcher la maquette pixel-perfect :
   - Consulter la doc du composant (`node scripts/get_component_docs.mjs NomComposant`)
   - Vérifier : `className`, `classNames` (slots), `styles`, `animation`, render props
   - Si un composant ne peut PAS être customisé pour ressembler à la maquette → le signaler comme construction manuelle
4. **Présenter le mapping à l'utilisateur** avec 2 sections :
   - ✅ **Composants librairie** : tel élément → tel composant HeroUI (customizable ✓)
   - 🔧 **Constructions manuelles** : tel élément → pas de composant dispo OU non customizable pour matcher la maquette, voici la contrainte
5. **Demander validation** UNIQUEMENT pour les constructions manuelles ou les cas douteux
   - Si le mapping a déjà été validé pour un pattern similaire dans le projet, ne pas redemander
   - L'utilisateur peut aider à trouver un composant sur le site de la librairie
6. **Ne coder qu'après validation** des constructions manuelles

**Règles :**
- La librairie UI est **flexible et customizable** — `className`, `classNames`, `styles`, `animation`
- **Ne construire manuellement QUE quand il n'y a AUCUN composant correspondant**
- **Si tu ne trouves pas le composant, DEMANDER** — l'utilisateur peut chercher sur le site

**Composants HeroUI Native disponibles (38) :**
Accordion, Alert, Avatar, BottomSheet, Button, Card, Checkbox, Chip, CloseButton, ControlField, Description, Dialog, FieldError, Input, InputGroup, InputOTP, Label, LinkButton, ListGroup, Menu, Popover, PressableFeedback, RadioGroup, ScrollShadow, SearchField, Select, Separator, Skeleton, SkeletonGroup, Slider, Spinner, Surface, Switch, Tabs, TagGroup, TextArea, TextField, Toast

**Correspondances systématiques (déjà validées) :**
| Élément maquette | Composant HeroUI |
|---|---|
| Item de liste avec fond | `Surface` ou `Card` |
| Menu/liste profil | `ListGroup` + `ListGroup.Item` + `Separator` |
| Séparateur/divider | `Separator` |
| Champ formulaire | `TextField` + `Input` + `Label` + `FieldError` |
| Lien/bouton texte | `LinkButton` + `LinkButton.Label` |
| Badge/tag statut | `Chip` |
| Checkbox | `Checkbox` + `Checkbox.Indicator` |
| Radio buttons | `RadioGroup` + `RadioGroup.Item` |
| Toggle | `Switch` |
| OTP code | `InputOTP` |
| Tabs/segmented | `Tabs` |
| Avatar | `Avatar` |
| Loader | `Spinner` ou `Skeleton` |
| Confirmation modale | `Dialog` |
| Barre de recherche | `SearchField` (sauf si API externe comme Google Places) |
| Toast/notification | `Toast` |

### Pas de MVP — tout implémenter complètement

- **Ne JAMAIS reporter une fonctionnalité à "plus tard"** — tout doit être opérationnel
- Intégrer les vrais services (Google Maps, API backend, push notifications, etc.) dès l'implémentation
- Ne pas utiliser de mocks/placeholders quand le vrai service est disponible
- Si une clé API ou un service est nécessaire, vérifier dans `.env` avant de demander
