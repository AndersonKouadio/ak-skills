# Interaction & Cursor Animations

## Cursor Trail (Example 1)

Image pool cycled with `gsap.ticker` + travel distance gate:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'

const IMAGES = ['/trail/flair-0.png', '/trail/flair-1.png', /* ... */]
const GAP = 100 // px between spawns

export function CursorTrail() {
  useGSAP(() => {
    const flair = gsap.utils.toArray<HTMLImageElement>('.flair')
    const wrapper = gsap.utils.wrap(0, flair.length)
    let index = 0
    let mousePos = { x: 0, y: 0 }
    let lastMousePos = { x: 0, y: 0 }
    let cachedMousePos = { x: 0, y: 0 }

    gsap.defaults({ duration: 1 })

    window.addEventListener('mousemove', (e) => {
      mousePos = { x: e.x, y: e.y }
    })

    function playAnimation(shape: HTMLImageElement) {
      const tl = gsap.timeline()
      tl.from(shape, { opacity: 0, scale: 0, ease: 'elastic.out(1,0.3)' })
        .to(shape, { rotation: 'random([-360, 360])' }, '<')
        .to(shape, { y: '120vh', ease: 'back.in(.4)', duration: 1 }, 0)
    }

    function animateImage() {
      const img = flair[wrapper(index)]
      gsap.killTweensOf(img)
      gsap.set(img, { clearProps: 'all' })
      gsap.set(img, {
        opacity: 1,
        left: mousePos.x,
        top: mousePos.y,
        xPercent: -50,
        yPercent: -50,
      })
      playAnimation(img)
      index++
    }

    gsap.ticker.add(() => {
      const dist = Math.hypot(lastMousePos.x - mousePos.x, lastMousePos.y - mousePos.y)

      cachedMousePos.x = gsap.utils.interpolate(cachedMousePos.x || mousePos.x, mousePos.x, 0.1)
      cachedMousePos.y = gsap.utils.interpolate(cachedMousePos.y || mousePos.y, mousePos.y, 0.1)

      if (dist > GAP) {
        animateImage()
        lastMousePos = { ...mousePos }
      }
    })
  })

  return (
    <div className="content">
      {IMAGES.map((src, i) => (
        <img key={i} className="flair" src={src} alt="" style={{ position: 'fixed', opacity: 0, width: 50 }} />
      ))}
    </div>
  )
}
```

---

## Cursor-Tracking Image Preview (Example 2)

`quickTo` for smooth tracking — image follows cursor when hovering a list item:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'

export function HoverPreviewList() {
  useGSAP(() => {
    gsap.set('.swipeimage', { yPercent: -50, xPercent: -50 })

    let firstEnter = true

    gsap.utils.toArray<HTMLElement>('.container').forEach((el) => {
      const image = el.querySelector<HTMLElement>('.swipeimage')!
      const setX = gsap.quickTo(image, 'x', { duration: 0.4, ease: 'power3' })
      const setY = gsap.quickTo(image, 'y', { duration: 0.4, ease: 'power3' })

      const align = (e: MouseEvent) => {
        if (firstEnter) {
          setX(e.clientX, e.clientX) // instant jump on first enter
          setY(e.clientY, e.clientY)
          firstEnter = false
        } else {
          setX(e.clientX)
          setY(e.clientY)
        }
      }

      const startFollow = () => document.addEventListener('mousemove', align)
      const stopFollow = () => document.removeEventListener('mousemove', align)

      const fade = gsap.to(image, {
        autoAlpha: 1,
        ease: 'none',
        paused: true,
        duration: 0.1,
        onReverseComplete: stopFollow,
      })

      el.addEventListener('mouseenter', (e) => {
        firstEnter = true
        fade.play()
        startFollow()
        align(e as MouseEvent)
      })
      el.addEventListener('mouseleave', () => fade.reverse())
    })
  })

  return (/* JSX */)
}
```

---

## Perspective Tilt (Example 10)

3D card tilt driven by cursor position:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'

export function PerspectiveTilt() {
  useGSAP(() => {
    gsap.set('.card-container', { perspective: 650 })

    const rotX = gsap.quickTo('.card', 'rotationX', { ease: 'power3' })
    const rotY = gsap.quickTo('.card', 'rotationY', { ease: 'power3' })
    const innerX = gsap.quickTo('.card-content', 'x', { ease: 'power3' })
    const innerY = gsap.quickTo('.card-content', 'y', { ease: 'power3' })

    const container = document.querySelector('.card-container')!

    container.addEventListener('pointermove', (e: Event) => {
      const { clientX, clientY } = e as PointerEvent
      rotX(gsap.utils.interpolate(15, -15, clientY / window.innerHeight))
      rotY(gsap.utils.interpolate(-15, 15, clientX / window.innerWidth))
      innerX(gsap.utils.interpolate(-30, 30, clientX / window.innerWidth))
      innerY(gsap.utils.interpolate(-30, 30, clientY / window.innerHeight))
    })

    container.addEventListener('pointerleave', () => {
      rotX(0); rotY(0); innerX(0); innerY(0)
    })
  })

  return (/* JSX with .card-container, .card, .card-content */)
}
```

---

## macOS Dock Effect (Example 3)

Proximity-based icon scaling with smooth cosine falloff:

```tsx
useGSAP(() => {
  const icons = document.querySelectorAll<HTMLElement>('.toolbarItem')
  const dock = document.querySelector<HTMLElement>('.toolbar')!
  const firstIcon = icons[0]

  const MIN = 48    // base icon size + margin
  const MAX = 120   // max enlarged size
  const BOUND = MIN * Math.PI

  gsap.set(icons, { transformOrigin: '50% 120%', height: 40 })
  gsap.set(dock, { position: 'relative', height: 60 })

  dock.addEventListener('mousemove', (e) => {
    const offset = dock.getBoundingClientRect().left + firstIcon.offsetLeft
    const pointer = e.clientX - offset

    icons.forEach((icon, i) => {
      const distance = i * MIN + MIN / 2 - pointer
      let x = 0, scale = 1

      if (-BOUND < distance && distance < BOUND) {
        const rad = distance / MIN * 0.5
        scale = 1 + (MAX / MIN - 1) * Math.cos(rad)
        x = 2 * (MAX - MIN) * Math.sin(rad)
      } else {
        x = (-BOUND < distance ? 2 : -2) * (MAX - MIN)
      }

      gsap.to(icon, { duration: 0.3, x, scale })
    })
  })

  dock.addEventListener('mouseleave', () => {
    gsap.to(icons, { duration: 0.3, scale: 1, x: 0 })
  })
})
```

---

## Draggable Plugin

```js
gsap.registerPlugin(Draggable, InertiaPlugin)

// Basic draggable
const [drag] = Draggable.create('.element', {
  type: 'x,y',                  // 'x', 'y', 'x,y', 'rotation', 'scroll'
  bounds: '.container',          // constrain to parent bounds
  inertia: true,                 // throw with momentum (requires InertiaPlugin)
  edgeResistance: 0.65,
  throwResistance: 3000,

  onPress() { /* pointer down */ },
  onDrag() {
    console.log(this.x, this.y)
  },
  onDragEnd() {
    console.log('ended at', this.endX, this.endY)
  },
  onThrowComplete() { /* inertia settled */ },

  // Snap on release
  snap: {
    x: (value) => Math.round(value / 50) * 50,
    y: (value) => Math.round(value / 50) * 50,
  },

  // Drag proxy — useful for touch on non-draggable containers
  trigger: '.drag-area',
})

// Enable/disable
drag.disable()
drag.enable()

// Drag proxy for seamless loop slider
Draggable.create('.drag-proxy', {
  type: 'x',
  trigger: '.cards',
  onPress() { this.startOffset = scrub.vars.offset },
  onDrag() {
    scrub.vars.offset = this.startOffset + (this.startX - this.x) * 0.001
    scrub.invalidate().restart()
  },
  onDragEnd() { scrollToOffset(scrub.vars.offset) }
})
```

---

## Observer Plugin

Unified touch/wheel/pointer/scroll event handler:

```js
gsap.registerPlugin(Observer)

Observer.create({
  target: window,
  type: 'wheel,touch,pointer',
  onUp: () => goToPrev(),
  onDown: () => goToNext(),
  tolerance: 10,           // min movement to trigger
  preventDefault: true,
})

// Full-screen section switching with Observer
let animating = false

Observer.create({
  type: 'wheel,touch',
  wheelSpeed: -1,
  onDown: () => !animating && goToSection(current - 1),
  onUp: () => !animating && goToSection(current + 1),
  tolerance: 10,
  preventDefault: true,
})

function goToSection(index) {
  index = gsap.utils.clamp(0, sections.length - 1, index)
  animating = true

  gsap.to(sections[index], {
    autoAlpha: 1,
    y: 0,
    duration: 0.8,
    ease: 'power3.inOut',
    onComplete: () => { animating = false }
  })
  gsap.to(sections[current], {
    autoAlpha: 0,
    y: -100,
    duration: 0.8,
    ease: 'power3.inOut',
  })
  current = index
}
```

---

## InertiaPlugin — Velocity Tracking

```js
gsap.registerPlugin(InertiaPlugin)

// Track element's velocity
InertiaPlugin.track('.element', 'x,y')

// Get current velocity
const vx = InertiaPlugin.getVelocity('.element', 'x')

// Animate with inertia (decelerate from current velocity)
gsap.to('.element', {
  inertia: {
    x: { velocity: 2000, end: 500, max: 800, min: 0 },
    y: { velocity: -500 },
  },
  onComplete: () => InertiaPlugin.untrack('.element'),
})
```
