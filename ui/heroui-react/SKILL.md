---
name: heroui-react
description: "HeroUI v3 React component library (Tailwind CSS v4 + React Aria). Use when building UIs with HeroUI — creating Buttons, Modals, Forms, Cards; installing @heroui/react; configuring dark/light themes with oklch variables; or fetching component docs. Keywords: HeroUI, Hero UI, heroui, @heroui/react, @heroui/styles."
metadata:
  author: heroui
  version: "3.0.0"
---

# HeroUI v3 React Development Guide

HeroUI v3 is a component library built on **Tailwind CSS v4** and **React Aria Components**, providing accessible, customizable UI components for React applications.

---

## Installation

```bash
curl -fsSL https://heroui.com/install | bash -s heroui-react
```

---

## CRITICAL: v3 Only - Ignore v2 Knowledge

**This guide is for HeroUI v3 ONLY.** Do NOT apply v2 patterns — the provider, styling, and component API all changed:

| Feature       | v2 (DO NOT USE)                   | v3 (USE THIS)                               |
| ------------- | --------------------------------- | ------------------------------------------- |
| Provider      | `<HeroUIProvider>` required       | **No Provider needed**                      |
| Animations    | `framer-motion` package           | CSS-based, no extra deps                    |
| Component API | Flat props: `<Card title="x">`    | Compound: `<Card><Card.Header>`             |
| Styling       | Tailwind v3 + `@heroui/theme`     | Tailwind v4 + `@heroui/styles`         	  |
| Packages      | `@heroui/system`, `@heroui/theme` | `@heroui/react`, `@heroui/styles` 		  |

```tsx
// DO NOT DO THIS - v2 pattern
import { HeroUIProvider } from "@heroui/react";
import { motion } from "framer-motion";

<HeroUIProvider>
	<Card title="Product" description="A great product" />
</HeroUIProvider>;
```

### CORRECT (v3 patterns)

```tsx
// DO THIS - v3 pattern (no provider, compound components)
import { Card } from "@heroui/react";

<Card>
	<Card.Header>
		<Card.Title>Product</Card.Title>
		<Card.Description>A great product</Card.Description>
	</Card.Header>
</Card>;
```

**Always fetch v3 docs before implementing.**

---

## Core Principles

- Semantic variants (`primary`, `secondary`, `tertiary`) over visual descriptions
- Composition over configuration (compound components)
- CSS variable-based theming with `oklch` color space
- BEM naming convention for predictable styling

---

## Accessing Documentation & Component Information

**For component details, examples, props, and implementation patterns, always fetch documentation:**

### Using Scripts

```bash
# List all available components
node scripts/list_components.mjs

# Get component documentation (MDX)
node scripts/get_component_docs.mjs Button
node scripts/get_component_docs.mjs Button Card TextField

# Get component source code
node scripts/get_source.mjs Button

# Get component CSS styles (BEM classes)
node scripts/get_styles.mjs Button

# Get theme variables
node scripts/get_theme.mjs

# Get non-component docs (guides, releases)
node scripts/get_docs.mjs /docs/react/getting-started/theming
```

### Direct MDX URLs

Component docs: `https://heroui.com/docs/react/components/{component-name}.mdx`

Examples:

- Button: `https://heroui.com/docs/react/components/button.mdx`
- Modal: `https://heroui.com/docs/react/components/modal.mdx`
- Form: `https://heroui.com/docs/react/components/form.mdx`

Getting started guides: `https://heroui.com/docs/react/getting-started/{topic}.mdx`

**Important:** Always fetch component docs before implementing. The MDX docs include complete examples, props, anatomy, and API references.

---

## Installation Essentials

### Quick Install

```bash
npm i @heroui/styles @heroui/react tailwind-variants
```

### Framework Setup (Next.js App Router - Recommended)

1. **Install dependencies:**

```bash
npm i @heroui/styles @heroui/react tailwind-variants tailwindcss @tailwindcss/postcss postcss
```

2. **Create/update `app/globals.css`:**

```css
/* Tailwind CSS v4 - Must be first */
@import "tailwindcss";

/* HeroUI v3 styles - Must be after Tailwind */
@import "@heroui/styles";
```

3. **Import in `app/layout.tsx`:**

```tsx
import "./globals.css";

export default function RootLayout({
	children,
}: {
	children: React.ReactNode;
}) {
	return (
		<html lang="en" suppressHydrationWarning>
			<body>
				{/* No Provider needed in HeroUI v3! */}
				{children}
			</body>
		</html>
	);
}
```

4. **Configure PostCSS (`postcss.config.mjs`):**

```js
export default {
	plugins: {
		"@tailwindcss/postcss": {},
	},
};
```

### Critical Setup Requirements

1. **Tailwind CSS v4 is MANDATORY** - HeroUI v3 will NOT work with Tailwind CSS v3
2. **Use Compound Components** - Components use compound structure (e.g., `Card.Header`, `Card.Content`)
3. **Use onPress, not onClick** - For better accessibility, use `onPress` event handlers
4. **Import Order Matters** - Always import Tailwind CSS before HeroUI styles

---

## Component Patterns

All components use the **compound pattern** shown above (dot-notation subcomponents like `Card.Header`, `Card.Content`). Don't flatten to props — always compose with subcomponents. Fetch component docs for complete anatomy and examples.

---

## Semantic Variants

HeroUI uses semantic naming to communicate functional intent:

| Variant     | Purpose                           | Usage          |
| ----------- | --------------------------------- | -------------- |
| `primary`   | Main action to move forward       | 1 per context  |
| `secondary` | Alternative actions               | Multiple       |
| `tertiary`  | Dismissive actions (cancel, skip) | Sparingly      |
| `danger`    | Destructive actions               | When needed    |
| `ghost`     | Low-emphasis actions              | Minimal weight |
| `outline`   | Secondary actions                 | Bordered style |

**Don't use raw colors** - semantic variants adapt to themes and accessibility.

---

## Theming

HeroUI v3 uses CSS variables with `oklch` color space:

```css
:root {
	--accent: oklch(0.6204 0.195 253.83);
	--accent-foreground: var(--snow);
	--background: oklch(0.9702 0 0);
	--foreground: var(--eclipse);
}
```

**Get current theme variables:**

```bash
node scripts/get_theme.mjs
```

**Color naming:**

- Without suffix = background (e.g., `--accent`)
- With `-foreground` = text color (e.g., `--accent-foreground`)

**Theme switching:**

```html
<html class="dark" data-theme="dark"></html>
```

For detailed theming, fetch: `https://heroui.com/docs/react/getting-started/theming.mdx`

---

## Workflow obligatoire : Mapper la maquette aux composants AVANT de coder

**Ce workflow est OBLIGATOIRE pour chaque ecran/feature/composant a implementer.**

### Etape 1 — Analyser la maquette
Identifier chaque element visuel (header, cards, listes, inputs, boutons, separateurs, tabs, badges, avatars, etc.)

### Etape 2 — Mapper chaque element au composant HeroUI v3
- Lister les composants : `node scripts/list_components.mjs`
- Chercher la doc : `node scripts/get_component_docs.mjs NomComposant`

### Etape 3 — Verifier la customisabilite
Pour chaque composant mappe, confirmer qu'il peut etre style pour matcher la maquette pixel-perfect :
- Consulter la doc (`node scripts/get_component_docs.mjs NomComposant`)
- Verifier : `className`, slots, render props
- Si un composant ne peut PAS etre customise → signaler comme construction manuelle

### Etape 4 — Presenter le mapping a l'utilisateur
Deux sections :
- ✅ **Composants librairie** : tel element → tel composant HeroUI (customizable ✓)
- 🔧 **Constructions manuelles** : tel element → pas de composant dispo OU non customizable, voici la contrainte

### Etape 5 — Demander validation
- UNIQUEMENT pour les constructions manuelles ou les cas douteux
- Si le mapping a deja ete valide pour un pattern similaire, ne pas redemander
- L'utilisateur peut aider a trouver un composant sur le site de la librairie

### Etape 6 — Ne coder qu'apres validation des constructions manuelles

### Correspondances systematiques (deja validees)

| Element maquette | Composant HeroUI v3 |
|---|---|
| Bouton action | `Button` (variant: primary/secondary/tertiary/danger/ghost/outline) |
| Lien texte / action low-emphasis | `Link` |
| Champ formulaire | `TextField` + `Input` + `Label` + `FieldError` |
| Zone texte multilignes | `TextArea` |
| Champ recherche | `SearchField` |
| Champ numerique | `NumberField` |
| Champ date | `DateField` / `DatePicker` |
| Champ heure | `TimeField` |
| Select / Dropdown | `Select` + `Select.Trigger` + `Select.Value` + `Select.Popover` + `ListBox` + `ListBox.Item` |
| Combobox / Autocomplete | `ComboBox` |
| Checkbox | `Checkbox` + `Checkbox.Indicator` |
| Groupe checkboxes | `CheckboxGroup` |
| Radio buttons | `RadioGroup` + `RadioGroup.Item` |
| Toggle switch | `Switch` |
| Tabs / onglets | `Tabs` |
| Card / Surface | `Card` + `Card.Header` + `Card.Content` |
| Modale | `Modal` |
| Dialog confirmation | `AlertDialog` |
| Drawer / panneau lateral | `Drawer` |
| Toast / notification | `toast()` / `toast.success()` / `toast.danger()` + `Toast.Provider` |
| Spinner chargement | `Spinner` |
| Skeleton loading | `Skeleton` |
| Avatar | `Avatar` |
| Badge / tag statut | `Chip` |
| Tag group | `TagGroup` + `TagGroup.Item` |
| Separateur / divider | `Separator` |
| Barre de progression | `ProgressBar` |
| Progression circulaire | `ProgressCircle` |
| Tooltip | `Tooltip` |
| Popover | `Popover` |
| Accordion / disclosure | `Accordion` ou `Disclosure` / `DisclosureGroup` |
| Pagination | `Pagination` |
| Table de donnees | `Table` |
| Breadcrumbs | `Breadcrumbs` |
| OTP input | `InputOTP` |
| Slider | `Slider` |
| Clavier raccourci | `Kbd` |
| Bouton fermer | `CloseButton` |
| Menu contextuel | `Dropdown` |
| Navbar / navigation | **SUPPRIME en v3** → construire avec `<nav>` + Tailwind + `Link`/`Button` |
| Code inline | **SUPPRIME en v3** → utiliser `<code>` HTML |
| Snippet | **SUPPRIME en v3** → construire manuellement |
| Image | **SUPPRIME en v3** → utiliser `<img>` ou Next.js `Image` |

---

## Pieges & Regles (lecons apprises)

### Button
- **PAS de prop `color`** → utiliser `variant` (`primary`, `secondary`, `tertiary`, `outline`, `ghost`, `danger`, `danger-soft`)
- **PAS de prop `size`** → utiliser `className` Tailwind (`text-sm px-3`, `text-lg px-6`)
- **PAS de prop `radius`** → utiliser `className` Tailwind (`rounded-full`, `rounded-lg`)
- **PAS de prop `isIconOnly`** → styler le bouton directement
- **`onPress`** au lieu de `onClick` sur tous les composants interactifs HeroUI
- **`isDisabled`** (pas `disabled`) sur les composants HeroUI

### Select (v3 compound pattern)
```tsx
// ❌ FAUX — v2 pattern
<Select label="Choix"><SelectItem key="a">Option A</SelectItem></Select>

// ✅ CORRECT — v3 compound
<Select>
  <Label>Choix</Label>
  <Select.Trigger><Select.Value /><Select.Indicator /></Select.Trigger>
  <Select.Popover>
    <ListBox>
      <ListBox.Item id="a" textValue="Option A">Option A</ListBox.Item>
    </ListBox>
  </Select.Popover>
</Select>
```

### Input / TextField (v3)
```tsx
// ❌ FAUX — v2 pattern
<Input label="Email" variant="bordered" isRequired labelPlacement="inside" />

// ✅ CORRECT — v3 compound ou Input seul
<TextField isRequired>
  <Label>Email</Label>
  <Input placeholder="email" type="email" />
  <FieldError />
</TextField>

// Ou Input seul (sans label/error)
<Input placeholder="email" type="email" required />
```

### Toast (v3)
```tsx
// ❌ FAUX — addToast n'existe pas
import { addToast } from "@heroui/toast";
addToast({ title: "Succes", color: "success" });

// ✅ CORRECT — toast() depuis @heroui/react
import { toast } from "@heroui/react";
toast("Message simple");
toast.success("Operation reussie");
toast.danger("Erreur survenue");
toast.warning("Attention");
toast.info("Information");

// Setup : ajouter Toast.Provider dans le layout
import { Toast } from "@heroui/react";
<Toast.Provider placement="top end" />
```

### Card (v3 compound)
```tsx
<Card>
  <Card.Header>
    <Card.Title>Titre</Card.Title>
    <Card.Description>Description</Card.Description>
  </Card.Header>
  <Card.Content>Contenu</Card.Content>
  <Card.Footer>Actions</Card.Footer>
</Card>
```

### Dropdown / Menu (v3)
```tsx
<Dropdown>
  <Dropdown.Trigger asChild>
    <Button>Menu</Button>
  </Dropdown.Trigger>
  <Dropdown.Content>
    <Dropdown.Item>Option 1</Dropdown.Item>
    <Dropdown.Item>Option 2</Dropdown.Item>
  </Dropdown.Content>
</Dropdown>
```

### Composants SUPPRIMES en v3
| v2 | Remplacement v3 |
|---|---|
| `Navbar` | `<nav>` HTML + Tailwind CSS |
| `Code` | `<code>` HTML |
| `Snippet` | Construction manuelle |
| `Image` | `<img>` ou Next.js `Image` |
| `User` | Construction manuelle |
| `Spacer` | `<div className="h-4">` ou gap Tailwind |
| `Ripple` | Effet CSS ou Button avec ripple |

### Styling
- **`className`** au lieu de `classNames` (v2 → v3)
- **Classes utilitaires** : `text-tiny` → `text-xs`, `text-small` → `text-sm`, `rounded-small` → `rounded-sm`
- **Couleurs** : `bg-primary` → `bg-accent` (systeme couleurs v3)
