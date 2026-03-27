# Asset Management

## Handling SVG Resources

### From External References (Codepen, Figma, etc.)

When a reference animation uses SVG assets from an external URL:

1. **Extract the SVG markup** from the source HTML
2. **Identify reusable parts**: gradients, paths, filters, patterns
3. **Save to project** at `public/svg/` or `src/assets/svg/`
4. **Adapt colors** to match project design tokens

```bash
# Example: download an SVG
curl -o public/svg/logo.svg "https://example.com/logo.svg"
```

### Inlining SVGs for GSAP Access

GSAP can **only animate inline SVG elements** — not `<img>` tags or CSS backgrounds.

```tsx
// ❌ NOT animatable by GSAP
<img src="/icon.svg" />

// ✅ GSAP can animate paths
<svg viewBox="0 0 100 100">
  <path className="animate-me" d="M..." />
</svg>

// ✅ With SVGR (auto-converts to React component)
import Logo from '@/assets/logo.svg'
<Logo /> // renders as inline SVG
```

### Setting up SVGR in Next.js

```bash
npm install @svgr/webpack
```

```js
// next.config.js
const nextConfig = {
  webpack(config) {
    config.module.rules.push({
      test: /\.svg$/,
      use: ['@svgr/webpack'],
    })
    return config
  }
}
```

### Adapting SVG Colors to Brand

```tsx
// Use currentColor for single-color SVGs
<svg fill="currentColor" className="text-brand-primary">
  <path d="..." />
</svg>

// Replace hardcoded colors with CSS variables
<path fill="var(--color-brand)" />

// Or use GSAP to set initial colors
gsap.set('.svg-element', { fill: 'var(--brand-color)' })
```

---

## Handling Images

### Downloading External Images to Project

When a reference uses images from external CDNs (e.g., assets.codepen.io):

```bash
# Download to public folder
curl -o public/images/hero.jpg "https://assets.codepen.io/16327/portrait-image-1.jpg"

# Batch download
for i in 1 2 3 4 5; do
  curl -o public/images/portrait-$i.jpg "https://assets.codepen.io/16327/portrait-image-$i.jpg"
done
```

### Next.js Image Component

Always use `next/image` for static images:

```tsx
import Image from 'next/image'

// Replace external URL src with local path after downloading
<Image
  src="/images/hero.jpg"
  alt="Hero"
  width={800}
  height={600}
  priority  // LCP image
  className="animate-target"  // GSAP selects this
/>
```

**Note**: `next/image` renders a wrapper `<span>` — animate the `<img>` inside or the wrapper, not the `Image` component:

```tsx
const imgRef = useRef<HTMLImageElement>(null)
// ...
<Image ref={imgRef} ... /> // ref goes to <img> element
gsap.from(imgRef.current, { opacity: 0 })
```

### For Image Sequences

Download all frames locally to avoid CORS and CDN latency:

```bash
# Apple-style image sequence
for i in $(seq -w 1 147); do
  curl -o public/sequence/frame-$i.jpg "https://cdn.example.com/frames/$i.jpg"
done
```

```js
// Reference locally
const urls = Array.from({ length: 147 }, (_, i) =>
  `/sequence/frame-${String(i + 1).padStart(4, '0')}.jpg`
)
```

---

## Handling Videos

When a reference animation should be reproduced from a video:

1. **Analyze the video** frame by frame — identify:
   - Which elements animate (text, images, shapes, backgrounds)
   - Timing and sequencing
   - Easing character (springy, smooth, snappy)
   - Scroll trigger points if applicable

2. **Map to GSAP concepts**:
   - Text entering from below → `SplitText` + `yPercent: 100` + `mask: 'lines'`
   - Shape morphing → `MorphSVGPlugin`
   - Layout rearranging → `Flip`
   - Parallax scrolling → `ScrollTrigger` + `scrub`

3. **Download reference assets** if needed

4. **Build the animation** matching timing

### Video as Background

```tsx
<video
  ref={videoRef}
  autoPlay muted loop playsInline
  className="hero-video"
  src="/videos/hero-bg.mp4"
/>
// GSAP can animate opacity, scale, position of the <video> element
```

---

## Font Management

Fonts must load before SplitText runs:

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] })

// In component:
useGSAP(() => {
  document.fonts.ready.then(() => {
    // Safe to run SplitText here
    SplitText.create('.headline', { ... })
  })
})
```

For custom fonts, add to `next/font/local` or load via `@font-face` in globals.css.

---

## Accessing Project Design Tokens for Animations

Read the project's design system before hardcoding animation colors:

```tsx
// Read CSS variables for use in GSAP
const brandColor = getComputedStyle(document.documentElement)
  .getPropertyValue('--color-brand').trim()

// Or use in GSAP directly (CSS vars are supported)
gsap.to('.el', { backgroundColor: 'var(--color-brand)' })

// For Tailwind projects, check tailwind.config.ts for color tokens
```

---

## Checklist Before Coding

- [ ] All referenced images downloaded to `public/`
- [ ] SVGs that need GSAP access are inline (not `<img>`)
- [ ] External SVG colors adapted to project design tokens
- [ ] Fonts loaded before SplitText
- [ ] Video assets in `public/videos/` if used as reference
- [ ] Large image sequences stored locally
