---
name: tailwindcss-v4
description: Tailwind CSS v4 — NE JAMAIS écrire du code Tailwind v3. Applique les conventions v4 (CSS-first, @theme, @import, oklch, renommages, container queries, logical properties). Chargé automatiquement dès que du Tailwind est écrit.
user-invocable: false
---

# Tailwind CSS v4 — Règles obligatoires

**Tu utilises Tailwind CSS v4. JAMAIS v3.** Si tu écris du code Tailwind, applique ces règles sans exception.

## Utilitaires renommés (v3 → v4)

| v3 (INTERDIT) | v4 (CORRECT) |
|---|---|
| `shadow-sm` | `shadow-xs` |
| `shadow` (sans suffixe) | `shadow-sm` |
| `drop-shadow-sm` | `drop-shadow-xs` |
| `drop-shadow` (sans suffixe) | `drop-shadow-sm` |
| `blur-sm` | `blur-xs` |
| `blur` (sans suffixe) | `blur-sm` |
| `backdrop-blur-sm` | `backdrop-blur-xs` |
| `backdrop-blur` (sans suffixe) | `backdrop-blur-sm` |
| `rounded-sm` | `rounded-xs` |
| `rounded` (sans suffixe) | `rounded-sm` |
| `outline-none` | `outline-hidden` |
| `ring` (sans valeur) | `ring-3` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` |
| `flex-shrink-*` | `shrink-*` |
| `flex-grow-*` | `grow-*` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `bg-gradient-to-b` | `bg-linear-to-b` |
| `transform-none` | `scale-none` (reset individuel) |

## Utilitaires supprimés

| v3 (INTERDIT) | v4 (CORRECT) |
|---|---|
| `bg-opacity-50` | `bg-black/50` (modificateur d'opacité) |
| `text-opacity-50` | `text-black/50` |
| `border-opacity-50` | `border-black/50` |
| `divide-opacity-50` | `divide-black/50` |
| `ring-opacity-50` | `ring-black/50` |
| `placeholder-opacity-50` | `placeholder-black/50` |

## Valeurs par défaut changées

| Propriété | v3 | v4 |
|---|---|---|
| `ring` largeur | 3px | 1px (utiliser `ring-3`) |
| `ring` couleur | `blue-500` | `currentColor` (spécifier la couleur) |
| `border` couleur | `gray-200` | `currentColor` (spécifier la couleur) |
| `divide` couleur | `gray-200` | `currentColor` (spécifier la couleur) |
| Placeholder couleur | `gray-400` | Texte courant à 50% opacité |
| Button cursor | `pointer` | `default` |
| `hover:` | Toujours appliqué | Seulement sur appareils hover (`@media (hover: hover)`) |

## Import et configuration

### INTERDIT (v3)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
```javascript
// tailwind.config.js — N'EXISTE PLUS
module.exports = {
  content: ["./src/**/*.{js,ts,jsx,tsx}"],
  theme: { extend: { colors: { ... } } }
}
```
```javascript
// postcss.config.mjs — OBSOLÈTE
export default { plugins: { "postcss-import": {}, tailwindcss: {}, autoprefixer: {} } };
```

### CORRECT (v4)
```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.55 0.2 250);
  --font-display: "Satoshi", sans-serif;
  --breakpoint-3xl: 1920px;
}
```
```javascript
// postcss.config.mjs — Un seul plugin, pas d'autoprefixer
export default { plugins: { "@tailwindcss/postcss": {} } };
```

## Couleurs : oklch obligatoire

```css
/* INTERDIT */
--color-blue: #3b82f6;
--color-blue: rgb(59, 130, 246);

/* CORRECT */
--color-blue: oklch(0.64 0.2 252.43);
```

## Custom utilities : @utility au lieu de @layer

```css
/* INTERDIT (v3) */
@layer utilities {
  .tab-4 { tab-size: 4; }
}
@layer components {
  .btn { border-radius: 0.5rem; padding: 0.5rem 1rem; }
}

/* CORRECT (v4) */
@utility tab-4 {
  tab-size: 4;
}
@utility btn {
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
}
```

## Valeurs arbitraires : parenthèses pour CSS vars

```html
<!-- INTERDIT (v3) -->
<div class="bg-[--brand-color]"></div>
<div class="grid-cols-[max-content,auto]"></div>

<!-- CORRECT (v4) -->
<div class="bg-(--brand-color)"></div>
<div class="grid-cols-[max-content_auto]"></div>
```

Crochets `[]` → parenthèses `()` pour les variables CSS.
Virgules → underscores dans grid-cols, grid-rows, object-position.

## Variant stacking : gauche à droite

```html
<!-- INTERDIT (v3 : droite à gauche) -->
<ul class="first:*:pt-0 last:*:pb-0">

<!-- CORRECT (v4 : gauche à droite) -->
<ul class="*:first:pt-0 *:last:pb-0">
```

## space-* et divide-* : préférer gap

Le sélecteur a changé (`:not(:last-child)` au lieu de `:not([hidden]) ~ :not([hidden])`).
Préférer `flex` + `gap` ou `grid` + `gap` :

```html
<!-- ÉVITER -->
<div class="space-y-4">

<!-- PRÉFÉRER -->
<div class="flex flex-col gap-4">
```

## Transitions : propriétés individuelles

```html
<!-- INTERDIT (v3) -->
<button class="transition-[opacity,transform]">

<!-- CORRECT (v4) -->
<button class="transition-[opacity,scale]">
```

Utiliser `scale`, `rotate`, `translate` au lieu de `transform`.

## theme() → var()

```css
/* INTERDIT (v3) */
.my-class {
  background-color: theme(colors.red.500);
}

/* CORRECT (v4) */
.my-class {
  background-color: var(--color-red-500);
}
```

## @reference pour Vue/Svelte/CSS Modules

```vue
<style>
  @reference "../../app.css";
  h1 {
    @apply text-2xl font-bold text-red-500;
  }
</style>
```

## Container queries (natif, pas de plugin)

```html
<div class="@container">
  <div class="grid grid-cols-1 @sm:grid-cols-3 @lg:grid-cols-4">
</div>
```

## Gradients avancés

```html
<!-- Angle custom -->
<div class="bg-linear-45 from-indigo-500 to-pink-500">
<!-- Interpolation oklch -->
<div class="bg-linear-to-r/oklch from-indigo-500 to-teal-400">
<!-- Conic / Radial -->
<div class="bg-conic from-red-600 to-red-600">
<div class="bg-radial-[at_25%_25%] from-white to-zinc-900">
<!-- Supprimer une étape via-none -->
<div class="bg-linear-to-r from-red-500 via-orange-400 to-yellow-400 dark:via-none dark:from-blue-500">
```

## 3D transforms

```html
<div class="perspective-distant">
  <div class="rotate-x-12 rotate-z-6 transform-3d">
</div>
```

## Nouvelles fonctionnalités

| Feature | Exemple |
|---------|---------|
| `not-*` variant | `not-hover:opacity-75` |
| `in-*` variant (sans `group`) | `in-hover:underline` |
| `@starting-style` | `starting:open:opacity-0` |
| `field-sizing` | `field-sizing-content` (auto-resize textarea) |
| Ombres empilables | `shadow-md inset-shadow-sm ring-1 inset-ring-1` |
| `nth-*` variants | `nth-3:bg-red-500` |
| Descendant variant | Style tous les descendants |

## Préprocesseurs CSS : non supportés

Sass, Less, Stylus ne fonctionnent **plus** avec Tailwind v4.
Tailwind v4 EST le préprocesseur.

## Navigateurs minimum

- Safari 16.4+
- Chrome 111+
- Firefox 128+

## Documentation officielle

En cas de doute, consulter : https://tailwindcss.com/docs

## Checklist

- [ ] Pas de `tailwind.config.js`
- [ ] `@import "tailwindcss"` (pas `@tailwind`)
- [ ] Couleurs en oklch
- [ ] `bg-linear-*` (pas `bg-gradient-*`)
- [ ] `shadow-xs`/`shadow-sm` (pas `shadow-sm`/`shadow`)
- [ ] `rounded-xs`/`rounded-sm` (pas `rounded-sm`/`rounded`)
- [ ] `ring-3` (pas `ring` seul)
- [ ] `outline-hidden` (pas `outline-none`)
- [ ] Opacité via `/50` (pas `bg-opacity-50`)
- [ ] Couleur explicite sur `border`, `ring`, `divide`
- [ ] `@utility` (pas `@layer utilities`)
- [ ] `var()` (pas `theme()`)
- [ ] Parenthèses `()` pour CSS vars (pas `[]`)
- [ ] gap > space-*
- [ ] PostCSS = juste `@tailwindcss/postcss`
