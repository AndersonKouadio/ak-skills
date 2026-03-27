# Cursor Effect Patterns

## Custom Cursor Component

Replaces the default cursor with a branded animated dot:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'
import { useRef } from 'react'

export function CustomCursor() {
  const cursorRef = useRef<HTMLDivElement>(null)
  const dotRef = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    const cursor = cursorRef.current!
    const dot = dotRef.current!

    const moveX = gsap.quickTo(cursor, 'x', { duration: 0.6, ease: 'power3' })
    const moveY = gsap.quickTo(cursor, 'y', { duration: 0.6, ease: 'power3' })
    const dotX  = gsap.quickTo(dot,    'x', { duration: 0.1 })
    const dotY  = gsap.quickTo(dot,    'y', { duration: 0.1 })

    window.addEventListener('mousemove', (e) => {
      moveX(e.clientX)
      moveY(e.clientY)
      dotX(e.clientX)
      dotY(e.clientY)
    })

    // Expand on hover of interactive elements
    document.querySelectorAll('a, button, [data-cursor="expand"]').forEach((el) => {
      el.addEventListener('mouseenter', () => {
        gsap.to(cursor, { scale: 2.5, duration: 0.3, ease: 'power2.out' })
      })
      el.addEventListener('mouseleave', () => {
        gsap.to(cursor, { scale: 1, duration: 0.3, ease: 'power2.out' })
      })
    })
  })

  return (
    <>
      {/* Outer ring — lags behind */}
      <div
        ref={cursorRef}
        style={{
          position: 'fixed',
          top: -20, left: -20,
          width: 40, height: 40,
          borderRadius: '50%',
          border: '1.5px solid currentColor',
          pointerEvents: 'none',
          zIndex: 9999,
          mixBlendMode: 'difference',
        }}
      />
      {/* Inner dot — snaps */}
      <div
        ref={dotRef}
        style={{
          position: 'fixed',
          top: -4, left: -4,
          width: 8, height: 8,
          borderRadius: '50%',
          background: 'currentColor',
          pointerEvents: 'none',
          zIndex: 9999,
        }}
      />
    </>
  )
}
```

---

## Magnetic Button Effect

Element "pulls" toward the cursor when nearby:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'
import { useRef } from 'react'

export function MagneticButton({ children }: { children: React.ReactNode }) {
  const btnRef = useRef<HTMLButtonElement>(null)

  useGSAP(() => {
    const btn = btnRef.current!
    const strength = 0.4

    btn.addEventListener('mousemove', (e) => {
      const rect = btn.getBoundingClientRect()
      const x = e.clientX - rect.left - rect.width / 2
      const y = e.clientY - rect.top - rect.height / 2

      gsap.to(btn, {
        x: x * strength,
        y: y * strength,
        duration: 0.3,
        ease: 'power3.out',
      })
    })

    btn.addEventListener('mouseleave', () => {
      gsap.to(btn, { x: 0, y: 0, duration: 0.5, ease: 'elastic.out(1, 0.4)' })
    })
  }, { scope: btnRef })

  return (
    <button ref={btnRef} style={{ display: 'inline-block' }}>
      {children}
    </button>
  )
}
```

---

## Cursor Trail — Image Pool (Example 1)

See [interaction.md](../interaction.md) for full implementation.

Key pattern:
- Pool of images cycling with `gsap.utils.wrap`
- `gsap.ticker` checks travel distance each frame
- Only spawns image when mouse moves more than `GAP` pixels
- Each spawn: `gsap.killTweensOf(img)` → `gsap.set(clearProps)` → animate

---

## Hover Preview Image (Example 2)

See [interaction.md](../interaction.md) for full implementation.

Key pattern:
- `gsap.quickTo` for x/y tracking (smooth but responsive)
- Pass `(value, startValue)` to `quickTo` for instant jump on first enter
- `autoAlpha` fade handled by a paused tween (`.play()` / `.reverse()`)
- Event listener added/removed to avoid memory leaks

---

## Perspective Tilt Card (Example 10)

See [interaction.md](../interaction.md) for full implementation.

Key pattern:
- `gsap.set(parent, { perspective: 650 })`
- 4 `quickTo` instances: rotationX, rotationY, inner x, inner y
- `gsap.utils.interpolate(min, max, ratio)` maps normalized mouse position to rotation range
- All reset to 0 on `pointerleave`

---

## Text Scramble on Hover

```tsx
import { ScrambleTextPlugin } from 'gsap/ScrambleTextPlugin'
gsap.registerPlugin(ScrambleTextPlugin)

useGSAP(() => {
  document.querySelectorAll('.scramble').forEach((el) => {
    const original = el.textContent!

    el.addEventListener('mouseenter', () => {
      gsap.to(el, {
        duration: 0.8,
        scrambleText: {
          text: original,
          chars: 'upperCase',
          speed: 0.6,
          revealDelay: 0.3,
        }
      })
    })
  })
}, { scope: container })
```

---

## Blob Cursor (Canvas-based)

```tsx
useGSAP(() => {
  const canvas = document.querySelector<HTMLCanvasElement>('.blob-canvas')!
  const ctx = canvas.getContext('2d')!
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight

  const blob = { x: window.innerWidth / 2, y: window.innerHeight / 2, radius: 80 }

  const moveX = gsap.quickTo(blob, 'x', { duration: 0.8, ease: 'power3' })
  const moveY = gsap.quickTo(blob, 'y', { duration: 0.8, ease: 'power3' })

  window.addEventListener('mousemove', (e) => {
    moveX(e.clientX)
    moveY(e.clientY)
  })

  gsap.ticker.add(() => {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    const gradient = ctx.createRadialGradient(blob.x, blob.y, 0, blob.x, blob.y, blob.radius)
    gradient.addColorStop(0, 'rgba(var(--brand-rgb), 0.3)')
    gradient.addColorStop(1, 'rgba(var(--brand-rgb), 0)')
    ctx.fillStyle = gradient
    ctx.beginPath()
    ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2)
    ctx.fill()
  })
}, { scope: container })
```
