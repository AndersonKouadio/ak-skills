# Flip Plugin

Flip animates elements smoothly between two layout states — even when their size, position, or parent changes in the DOM.

## Core Pattern: getState → from / to

```js
gsap.registerPlugin(Flip)

// 1. Capture current state (before DOM change)
const state = Flip.getState('.item')

// 2. Make your DOM/class change
element.classList.toggle('expanded')

// 3. Animate FROM the captured state TO the new layout
Flip.from(state, {
  duration: 0.6,
  ease: 'power2.inOut',
  absolute: true,    // use position:absolute during animation
  nested: true,      // handle nested elements
  prune: true,       // skip elements that don't move
  onEnter: (elements) => gsap.from(elements, { opacity: 0, scale: 0.8 }),
  onLeave: (elements) => gsap.to(elements, { opacity: 0, scale: 0.8 }),
})

// OR: Animate TO a target state (element must already be in new position)
Flip.to(state, { duration: 0.6, ease: 'power2.inOut' })
```

---

## Fit — Animate to Match Another Element's Bounds

```js
const targetState = Flip.getState('.target-marker')

// Animate .canvas to match the position/size of .target-marker
Flip.fit('.canvas', targetState, {
  duration: 1,
  ease: 'none',
})
```

---

## Bento Gallery (Example 5)

Flip from compact bento to full-screen layout on scroll:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { Flip } from 'gsap/Flip'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'

gsap.registerPlugin(Flip, ScrollTrigger)

export function BentoGallery() {
  useGSAP(() => {
    let flipCtx: gsap.Context | undefined

    const createTween = () => {
      const gallery = document.querySelector('#gallery') as HTMLElement
      const items = gallery.querySelectorAll('.gallery__item')

      flipCtx?.revert()
      gallery.classList.remove('gallery--final')

      flipCtx = gsap.context(() => {
        // Capture final state
        gallery.classList.add('gallery--final')
        const flipState = Flip.getState(items)
        gallery.classList.remove('gallery--final')

        const flip = Flip.to(flipState, {
          simple: true,
          ease: 'expoScale(1, 5)',
        })

        const tl = gsap.timeline({
          scrollTrigger: {
            trigger: gallery,
            start: 'center center',
            end: '+=100%',
            scrub: true,
            pin: gallery.parentElement,
          }
        })
        tl.add(flip)

        return () => gsap.set(items, { clearProps: 'all' })
      })
    }

    createTween()
    window.addEventListener('resize', createTween)
    return () => window.removeEventListener('resize', createTween)
  })

  return (/* JSX */)
}
```

---

## Card Stack (Example 7)

DOM manipulation + Flip for smooth card cycling:

```tsx
function moveCard() {
  const slider = document.querySelector('.slider')
  const lastItem = slider.querySelector('.item:last-child') as HTMLImageElement
  if (!slider || !lastItem) return

  lastItem.style.display = 'none'
  const newItem = document.createElement('img')
  newItem.className = lastItem.className
  newItem.src = lastItem.src
  slider.insertBefore(newItem, slider.firstChild)
}

document.body.addEventListener('click', () => {
  const state = Flip.getState('.item')
  moveCard()

  Flip.from(state, {
    targets: '.item',
    ease: 'sine.inOut',
    absolute: true,
    onEnter: (elements) =>
      gsap.from(elements, { duration: 0.3, yPercent: 20, opacity: 0, ease: 'expo.out' }),
    onLeave: (elements) =>
      gsap.to(elements, {
        duration: 0.3,
        yPercent: 5,
        xPercent: -5,
        transformOrigin: 'bottom left',
        opacity: 0,
        ease: 'expo.out',
        onComplete() {
          slider.removeChild(elements[0])
        }
      }),
  })
})
```

---

## Three.js + Flip Waypoints (Example 24)

Use `Flip.fit()` to move a canvas element between DOM markers while animating its contents:

```js
function buildTimeline() {
  ctx?.revert()
  ctx = gsap.context(() => {
    const s2 = Flip.getState('.second .marker')
    const s3 = Flip.getState('.third .marker')

    const tl = gsap.timeline({
      scrollTrigger: { start: 0, end: 'max', scrub: 2 }
    })

    // Hop to second + rotate mesh
    tl.add(Flip.fit(canvasEl, s2, { duration: 1, ease: 'none' }), 0)
      .to(mesh.rotation, { x: `+=${Math.PI}`, y: `+=${Math.PI}`, duration: 1, ease: 'none' }, '<')
      .addLabel('mid', '+=0.5')
      // Hop to third + rotate again
      .add(Flip.fit(canvasEl, s3, { duration: 1, ease: 'none' }), 'mid')
      .to(mesh.rotation, { x: `+=${Math.PI}`, y: `+=${Math.PI}`, duration: 1, ease: 'none' }, '<')
  })
}
```

---

## getState Options

```js
Flip.getState('.items', {
  props: 'opacity,backgroundColor',  // also track these CSS props
  simple: false,                      // don't include scale/rotation
})
```

---

## Flip.batch() — Many Elements

```js
// Batch Flip for performance with many elements
const batch = Flip.batch()
batch.add(Flip.getState('.cards'))
// ... DOM change ...
batch.animate({ duration: 0.5, ease: 'power2.inOut' })
```

---

## Common Pitfalls

| Issue | Solution |
|---|---|
| Elements jump on resize | Always `ctx.revert()` before recreating |
| Flip misaligns with transforms | Use `gsap.set(el, { clearProps: 'all' })` before capturing |
| `onEnter` elements start visible | Always set initial opacity: 0 before Flip.from |
| Canvas/Three.js doesn't resize correctly | `Flip.fit()` + manual `renderer.setSize()` after resize |
