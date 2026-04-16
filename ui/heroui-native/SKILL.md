---
name: heroui-native
description: "HeroUI Native component library for React Native (Tailwind v4 via Uniwind). Use when building mobile UIs with HeroUI Native — creating Buttons, Cards, TextFields, Dialogs; installing heroui-native; configuring dark/light themes; or fetching component docs. Keywords: HeroUI Native, heroui-native, React Native UI, Uniwind, mobile components."
metadata:
  author: heroui
  version: "2.0.0"
---

# HeroUI Native Development Guide

HeroUI Native is a component library built on **Uniwind (Tailwind CSS for React Native)** and **React Native**, providing accessible, customizable UI components for mobile applications.

---

## CRITICAL: Native Only - Do Not Use Web Patterns

| Feature      | React (Web)          | Native (Mobile)                     |
| ------------ | -------------------- | ----------------------------------- |
| **Styling**  | Tailwind CSS v4      | Uniwind (Tailwind for React Native) |
| **Colors**   | oklch format         | oklch format (via CSS vars)         |
| **Package**  | `@heroui/react`      | `heroui-native`                     |
| **Platform** | Web browsers         | iOS & Android                       |

**Always fetch Native docs before implementing.**

---

## Accessing Documentation

```bash
# List all available components
node scripts/list_components.mjs

# Get component documentation
node scripts/get_component_docs.mjs Button Card TextField

# Get theme variables
node scripts/get_theme.mjs

# Get non-component docs
node scripts/get_docs.mjs /docs/native/getting-started/theming
```

Direct MDX: `https://heroui.com/docs/native/components/{component-name}.mdx`

---

## Provider Setup

### Provider Hierarchy
```
HeroUINativeProvider
  SafeAreaListener
    GlobalAnimationSettingsProvider
      TextComponentProvider
        ToastProvider (conditionally rendered)
          Your App
          PortalHost (for overlays)
```

### Expo Router Integration
```tsx
// app/_layout.tsx
import { HeroUINativeProvider, type HeroUINativeConfig } from 'heroui-native';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { Stack } from 'expo-router';

const config: HeroUINativeConfig = {
  textProps: { minimumFontScale: 0.5, maxFontSizeMultiplier: 1.5 },
};

export default function RootLayout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <HeroUINativeProvider config={config}>
        <Stack />
      </HeroUINativeProvider>
    </GestureHandlerRootView>
  );
}
```

### Provider Rules
- **Single instance** — NEVER nest multiple `HeroUINativeProvider`
- **Config outside component** — define `const config` outside to avoid re-creation on render
- **PortalHost** is included automatically — no manual setup needed
- **Toast** can be disabled: `toast: false` or `toast: 'disabled'`
- **Global animation disable**: `animation: 'disable-all'`

---

## Composition & Compound Components

### Compound Pattern (dot notation)
```tsx
<Dialog>
  <Dialog.Trigger asChild>
    <Button variant="primary">Open</Button>
  </Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay />
    <Dialog.Content>
      <Dialog.Close />
      <Dialog.Title>Title</Dialog.Title>
      <Dialog.Description>Description</Dialog.Description>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog>
```

### asChild Prop
When `asChild={true}`, the component clones its child and merges props instead of rendering a wrapper.
```tsx
<Dialog.Trigger asChild>
  <Button variant="primary">Open Dialog</Button>
</Dialog.Trigger>
```

### Text color on Button
Text color classes go on `Button.Label`, NOT on `Button`:
```tsx
<Button className="bg-accent">
  <Button.Label className="text-accent-foreground">Click</Button.Label>
</Button>
```

---

## Portal — State Management Critical Rule

**Interactive state inside portals MUST live in a separate child component**, not the parent.

```tsx
// ❌ FAUX — state in parent causes issues
function Parent() {
  const [dialogOpen, setDialogOpen] = useState(false);
  const [inputValue, setInputValue] = useState(""); // ← bug!
  return (
    <Dialog isOpen={dialogOpen} onOpenChange={setDialogOpen}>
      <Dialog.Portal>
        <Input value={inputValue} onChangeText={setInputValue} />
      </Dialog.Portal>
    </Dialog>
  );
}

// ✅ CORRECT — separate component for portal content
function Parent() {
  const [dialogOpen, setDialogOpen] = useState(false);
  return (
    <Dialog isOpen={dialogOpen} onOpenChange={setDialogOpen}>
      <Dialog.Portal>
        <DialogFormContent onClose={() => setDialogOpen(false)} />
      </Dialog.Portal>
    </Dialog>
  );
}

function DialogFormContent({ onClose }) {
  const [inputValue, setInputValue] = useState("");
  return (
    <Dialog.Content>
      <Input value={inputValue} onChangeText={setInputValue} />
    </Dialog.Content>
  );
}
```

---

## Colors & Theming

### Color System (oklch via CSS variables)
- Colors without suffix = backgrounds (`--accent`)
- Colors with `-foreground` = text (`--accent-foreground`)
- Use `bg-accent` / `text-accent-foreground` in className

### Theme-Invariant Primitives
`--white`, `--black`, `--snow`, `--eclipse` — same in light and dark.

### Override Colors in global.css
```css
@layer theme {
  @variant light {
    --accent: oklch(0.65 0.25 270);
  }
  @variant dark {
    --accent: oklch(0.65 0.25 270);
  }
}
```

### Add Custom Semantic Colors
```css
@layer theme {
  @variant light { --info: oklch(0.6 0.15 210); --info-foreground: oklch(0.98 0 0); }
  @variant dark  { --info: oklch(0.7 0.12 210); --info-foreground: oklch(0.15 0 0); }
}
@theme inline {
  --color-info: var(--info);
  --color-info-foreground: var(--info-foreground);
}
```

### useThemeColor Hook
```tsx
import { useThemeColor } from 'heroui-native';

// Single
const accent = useThemeColor('accent');

// Multiple (tuple — more efficient)
const [accent, bg, danger] = useThemeColor(['accent', 'background', 'danger']);
```

### Theme Switching
```tsx
import { Uniwind, useUniwind } from 'uniwind';
const { theme } = useUniwind();
Uniwind.setTheme(theme === 'light' ? 'dark' : 'light');
```

### Custom Fonts
```css
@theme {
  --font-normal: 'YourFont-400Regular';
  --font-medium: 'YourFont-500Medium';
  --font-semibold: 'YourFont-600SemiBold';
}
```
Font names must match PostScript names. All HeroUI components use these automatically.

### Form Field Variables
Form controls use `--field-*` variables. Update these to restyle inputs/checkboxes without impacting buttons/cards:
`--field-background`, `--field-foreground`, `--field-placeholder`, `--field-border`, `--field-border-width`

---

## Styling

### Three Approaches
1. `className` — Tailwind via Uniwind
2. `style` — StyleSheet API (takes precedence over className)
3. Render props — dynamic styling based on state

### Style Precedence
- `style` prop > `className`
- Animated properties (via reanimated) > `className`
- Modify animated styles via `animation` prop, not `className`

### cn Utility (class merging)
```tsx
import { cn } from 'heroui-native';
cn('bg-background p-4', 'bg-accent'); // → 'p-4 bg-accent' (conflict resolved)
cn('text-sm', isActive && 'text-accent font-bold', className);
```

### classNames Exports (reuse component styles)
```tsx
import { buttonClassNames } from 'heroui-native';
const rootClasses = buttonClassNames.root({ variant: 'primary', size: 'md' });
const labelClasses = buttonClassNames.label({ variant: 'primary', size: 'md' });
```
Available: `buttonClassNames`, `cardClassNames`, `chipClassNames`, etc.

### Custom Variants with tailwind-variants
```tsx
import { tv, type VariantProps } from 'tailwind-variants';
const customButton = tv({
  base: 'font-semibold rounded-lg',
  variants: { intent: { primary: 'bg-blue-500', danger: 'bg-red-500' } },
  defaultVariants: { intent: 'primary' },
});
```

---

## Animation

### Central `animation` Prop
```tsx
// Customize values
<Switch animation={{ scale: { value: [1, 0.9] }, backgroundColor: { value: ['#172554', '#eab308'] } }}>
  <Switch.Thumb />
</Switch>

// Spring config
<Accordion.Indicator animation={{ rotation: { springConfig: { damping: 60, stiffness: 900 } } }} />

// Layout animations
<Accordion.Content animation={{ entering: { value: FadeInRight.delay(50) } }} />
```

### Disabling Animations
- Component level: `animation={false}` or `animation="disabled"`
- Root cascade (ALL children): `animation="disable-all"`
- Global: `<HeroUINativeProvider config={{ animation: 'disable-all' }}>`

### Accessibility — Reduce Motion
Handled automatically. Library respects `useReducedMotion()` — no manual handling needed.

---

## Pièges & Règles (leçons apprises)

### Button

- **PAS de prop `color`** → utiliser `variant` (`primary`, `secondary`, `tertiary`, `outline`, `ghost`, `danger`, `danger-soft`)
- **PAS de prop `isLoading`** → utiliser `isDisabled` + `<Spinner />` manuellement
- **PAS de `variant="flat"`** → utiliser `secondary` ou `ghost`
- **LinkButton** pour les liens inline (ghost sans highlight) : `<LinkButton><LinkButton.Label>Texte</LinkButton.Label></LinkButton>`

```tsx
// ❌ FAUX
<Button color="primary" isLoading={loading}>Valider</Button>

// ✅ CORRECT
<Button variant="primary" isDisabled={loading}>
  {loading ? <Spinner size="sm" /> : <Button.Label>Valider</Button.Label>}
</Button>
```

### BottomSheet — Scrollable Content

- `BottomSheet.Content` wraps les children dans un `BottomSheetView` (NON scrollable)
- **`BottomSheetFlatList` ne scrolle PAS dans `BottomSheet.Content`**
- **Pour du contenu scrollable** : utiliser `GorhomBottomSheet` directement

```tsx
// ✅ CORRECT — GorhomBottomSheet pour contenu scrollable
import GorhomBottomSheet, { BottomSheetFlatList, BottomSheetBackdrop } from '@gorhom/bottom-sheet';

<GorhomBottomSheet ref={sheetRef} index={-1} snapPoints={['60%']}
  enableDynamicSizing={false} enablePanDownToClose
  backdropComponent={renderBackdrop}
  onChange={(index) => { if (index === -1) onClose(); }}
>
  <BottomSheetFlatList data={items} keyExtractor={(i) => i.id} renderItem={renderItem} />
</GorhomBottomSheet>
```

### BottomSheet — Z-Index & overflow:hidden

- **GorhomBottomSheet DOIT être rendu HORS des containers `overflow: hidden`**
- Remonter le state et le BottomSheet au niveau écran parent
- Sinon le BottomSheet sera clippé/masqué

### BottomSheet — API HeroUI (contenu statique)

- Compound : `BottomSheet > BottomSheet.Portal > BottomSheet.Overlay + BottomSheet.Content`
- **PAS de `onClose`** → utiliser `onOpenChange`
- `snapPoints` sur `BottomSheet.Content`, pas le root

### Checkbox

- **`onSelectedChange`** (pas `onValueChange`)
- Style conditionnel : transparent par défaut, accent au check

```tsx
<Checkbox
  isSelected={val}
  onSelectedChange={setVal}
  className={val ? 'bg-accent border-accent' : 'bg-transparent border-default'}
>
  <Checkbox.Indicator className={val ? 'bg-accent' : ''} />
</Checkbox>
```

### Forms — TextField + Input (pas TextInput natif)

- **Toujours `TextField` + `Input` + `Label` + `FieldError`** de HeroUI
- **JAMAIS `TextInput` natif** de React Native
- `Input` supporte : `keyboardType`, `secureTextEntry`, `autoCapitalize`, `maxLength`, `multiline`, `variant`

```tsx
<Controller control={form.control} name="email"
  render={({ field: { onChange, onBlur, value } }) => (
    <TextField isInvalid={!!errors.email}>
      <Label>Email</Label>
      <Input placeholder="email" keyboardType="email-address" value={value}
        onChangeText={onChange} onBlur={onBlur} />
      <FieldError>{errors.email?.message}</FieldError>
    </TextField>
  )}
/>
```

### Input dans BottomSheet

```tsx
import { useBottomSheetAwareHandlers } from 'heroui-native';
const { onFocus, onBlur } = useBottomSheetAwareHandlers();
<Input onFocus={onFocus} onBlur={onBlur} />
```

### Couleurs sur fond accent — useThemeColor

- **`useThemeColor("#FFFFFF")` NE FONCTIONNE PAS** — cherche `--color-#FFFFFF`, retourne `""`
- Le `??` ne rattrape pas `""` (truthy pour `??`)
- **Composant Icon custom DOIT détecter hex/rgb et bypasser useThemeColor**

```tsx
const isHexOrRgb = (c: string) => c.startsWith('#') || c.startsWith('rgb');
const finalColor = isHexOrRgb(color) ? color : useThemeColor(color);
```

### KeyboardAvoidingView

- **Pas nécessaire** — `ScrollView` + `keyboardShouldPersistTaps="handled"` suffit
- Peut causer des conflits avec BottomSheets gorhom
