# GSAP in React / Next.js

## The Golden Rule

**Always use `useGSAP()` hook.** Never put GSAP code in `useEffect` directly — it skips GSAP's scoping system and creates memory leaks.

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { useRef } from 'react'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(useGSAP, ScrollTrigger)

export function AnimatedSection() {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    // All GSAP code goes here — auto-cleaned on unmount
    gsap.from('.card', {
      y: 60,
      opacity: 0,
      stagger: 0.1,
      scrollTrigger: {
        trigger: container.current,
        start: 'top 80%',
      }
    })
  }, { scope: container })

  return <div ref={container}><div className="card">...</div></div>
}
```

---

## Installation

```bash
npm install gsap @gsap/react
```

For Club plugins (SplitText, ScrollSmoother, MorphSVG, Flip, etc.):
```bash
# Install from GSAP's private registry or copy from downloaded ZIP
npm install gsap@npm:@gsap/shockingly  # Club
```

---

## Plugin Registration

Register once, globally — typically in a layout or providers file:

```tsx
// app/providers.tsx or lib/gsap-init.ts
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { ScrollSmoother } from 'gsap/ScrollSmoother'
import { SplitText } from 'gsap/SplitText'
import { Flip } from 'gsap/Flip'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import { DrawSVGPlugin } from 'gsap/DrawSVGPlugin'
import { MotionPathPlugin } from 'gsap/MotionPathPlugin'
import { Draggable } from 'gsap/Draggable'
import { InertiaPlugin } from 'gsap/InertiaPlugin'
import { Observer } from 'gsap/Observer'
import { CustomEase } from 'gsap/CustomEase'
import { useGSAP } from '@gsap/react'

gsap.registerPlugin(
  useGSAP,
  ScrollTrigger,
  ScrollSmoother,
  SplitText,
  Flip,
  MorphSVGPlugin,
  DrawSVGPlugin,
  MotionPathPlugin,
  Draggable,
  InertiaPlugin,
  Observer,
  CustomEase,
)
```

---

## SSR Safety

Next.js runs components on the server. Some GSAP code needs browser APIs.

```tsx
// Safe — useGSAP only runs client-side
useGSAP(() => {
  // This is always safe
}, { scope: container })

// If you need window/document outside of useGSAP:
const isBrowser = typeof window !== 'undefined'

// For dynamic imports of heavy plugins:
useEffect(() => {
  import('gsap/ScrollSmoother').then(({ ScrollSmoother }) => {
    gsap.registerPlugin(ScrollSmoother)
  })
}, [])
```

Always add `'use client'` to any component using GSAP.

---

## App Router Layout — ScrollSmoother Structure

ScrollSmoother requires a specific DOM structure. In Next.js App Router:

```tsx
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <div id="smooth-wrapper">
          <div id="smooth-content">
            {children}
          </div>
        </div>
      </body>
    </html>
  )
}

// app/providers.tsx (Client Component)
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'
import { ScrollSmoother } from 'gsap/ScrollSmoother'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollSmoother, ScrollTrigger)

export function GSAPProvider({ children }) {
  useGSAP(() => {
    ScrollSmoother.create({
      wrapper: '#smooth-wrapper',
      content: '#smooth-content',
      smooth: 2,
      effects: true,
      normalizeScroll: true,
    })
  })

  return <>{children}</>
}
```

---

## ScrollTrigger + Next.js Router

When navigating between pages, ScrollTrigger instances from the previous page can persist. Always kill them on route change:

```tsx
'use client'
import { usePathname } from 'next/navigation'
import { useEffect } from 'react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

export function ScrollTriggerCleaner() {
  const pathname = usePathname()

  useEffect(() => {
    return () => {
      ScrollTrigger.getAll().forEach(t => t.kill())
      ScrollTrigger.clearScrollMemory()
    }
  }, [pathname])

  return null
}
```

---

## Font Loading Before SplitText

SplitText must run AFTER fonts are loaded, or character widths will be wrong and lines will break incorrectly.

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const split = SplitText.create('.headline', {
      type: 'words,lines',
      mask: 'lines',
      autoSplit: true,
      onSplit: (instance) => {
        return gsap.from(instance.lines, {
          yPercent: 120,
          stagger: 0.08,
          duration: 1,
          ease: 'power3.out',
        })
      }
    })
  })
}, { scope: container })
```

---

## gsap.context() for Manual Cleanup

When `useGSAP` isn't available (class components, plain JS modules):

```tsx
let ctx: gsap.Context

function createAnimations() {
  ctx?.revert() // Clean up previous
  ctx = gsap.context(() => {
    // All animations here
    gsap.from('.item', { y: 40, opacity: 0, stagger: 0.1 })
    return () => {
      // Optional cleanup inside context
    }
  })
}

// On resize:
window.addEventListener('resize', createAnimations)
// On unmount:
ctx.revert()
```

---

## Performance Rules

1. **`will-change: transform`** — add to elements that will animate transform/opacity
2. **`force3D: true`** — for elements with many simultaneous transforms
3. **`gsap.quickSetter()`** — for high-frequency updates (mousemove, ticker)
4. **`gsap.quickTo()`** — for smooth cursor tracking (auto-tweens to target)
5. **`invalidateOnRefresh: true`** — on ScrollTrigger when layout changes
6. **Never animate `width`/`height`** — use `scaleX`/`scaleY` instead
7. **Batch DOM reads** — use `ScrollTrigger.batch()` for many elements

```tsx
// BAD — creates a new tween every mousemove
el.addEventListener('mousemove', (e) => {
  gsap.to(cursor, { x: e.clientX, y: e.clientY })
})

// GOOD — one persistent tween, just update target value
const moveX = gsap.quickTo(cursor, 'x', { duration: 0.4, ease: 'power3' })
const moveY = gsap.quickTo(cursor, 'y', { duration: 0.4, ease: 'power3' })
el.addEventListener('mousemove', (e) => {
  moveX(e.clientX)
  moveY(e.clientY)
})
```

---

## Responsive Animations with matchMedia

```tsx
useGSAP(() => {
  const mm = gsap.matchMedia()

  mm.add('(min-width: 768px)', () => {
    // Desktop animation
    gsap.from('.hero-title', { y: 100, opacity: 0, duration: 1 })
  })

  mm.add('(max-width: 767px)', () => {
    // Mobile animation (simpler)
    gsap.from('.hero-title', { opacity: 0, duration: 0.6 })
  })
}, { scope: container })
```

---

## Common Mistakes to Avoid

| Wrong | Right |
|---|---|
| `useEffect` with GSAP | `useGSAP()` |
| Selecting elements globally | `scope: containerRef` in `useGSAP` |
| Registering plugins inside component | Register globally once |
| Using SplitText before fonts load | Wait for `document.fonts.ready` |
| Creating ScrollSmoother in useGSAP | Create it in a top-level layout provider |
| Not calling `ctx.revert()` on resize | Always revert context before recreating |
| `gsap.to(element)` in render/JSX | Only in `useGSAP` or event handlers |
