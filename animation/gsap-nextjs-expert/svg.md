# SVG Animations

## MorphSVGPlugin (Club)

Morphs one SVG path into another. Handles paths with different number of points.

```js
gsap.registerPlugin(MorphSVGPlugin)

gsap.to('#path', {
  morphSVG: '#targetPath',    // selector, element, or path data string
  duration: 1,
  ease: 'power2.inOut',
})

gsap.to('#path', {
  morphSVG: 'M 0 100 V 0 Q 50 0 100 0 V 100 z',  // raw path data
  duration: 0.8,
})

// type: 'rotational' | 'linear' (default: rotational — usually smoother)
gsap.to('#path', {
  morphSVG: { shape: '#target', type: 'rotational', origin: '50% 50%' }
})
```

---

## Curve Swipe / Page Transition (Example 11)

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import gsap from 'gsap'

gsap.registerPlugin(MorphSVGPlugin)

export function CurveTransition() {
  useGSAP(() => {
    const path = document.querySelector('.path')
    const start = 'M 0 100 V 50 Q 50 0 100 50 V 100 z'
    const end   = 'M 0 100 V 0 Q 50 0 100 0 V 100 z'

    const tl = gsap.timeline()
    tl.to(path, { morphSVG: start, ease: 'power2.in' })
      .to(path, { morphSVG: end,   ease: 'power2.out' })
      .reverse()

    document.body.addEventListener('click', () => {
      tl.reversed(!tl.reversed())
    })
  })

  return (
    <svg className="transition" viewBox="0 0 100 100" preserveAspectRatio="xMidYMin slice">
      <path className="path" d="M 0 100 V 100 Q 50 100 100 100 V 100 z" />
    </svg>
  )
}
```

SVG must use `preserveAspectRatio="none"` for full-screen fills, or `xMidYMin slice` for viewport-filling.

---

## Dynamic Wave Overlay (Example 12)

Animate control points of bezier curves for organic morphing:

```js
const allPoints = [] // [pathIndex][pointIndex] = value (0-100)

// Initialize all points at 100 (bottom)
for (let i = 0; i < numPaths; i++) {
  allPoints.push(Array(numPoints).fill(100))
}

function toggle() {
  tl.progress(0).clear()
  for (let i = 0; i < numPaths; i++) {
    const pathDelay = delayPerPath * (isOpened ? i : numPaths - i - 1)
    for (let j = 0; j < numPoints; j++) {
      const delay = Math.random() * 0.3
      tl.to(allPoints[i], { [j]: 0 }, delay + pathDelay)
    }
  }
}

function render() {
  paths.forEach((path, i) => {
    const points = allPoints[i]
    let d = isOpened
      ? `M 0 0 V ${points[0]} C`
      : `M 0 ${points[0]} C`

    for (let j = 0; j < numPoints - 1; j++) {
      const p = (j + 1) / (numPoints - 1) * 100
      const cp = p - (1 / (numPoints - 1) * 100) / 2
      d += ` ${cp} ${points[j]} ${cp} ${points[j + 1]} ${p} ${points[j + 1]}`
    }
    d += isOpened ? ' V 100 H 0' : ' V 0 H 0'
    path.setAttribute('d', d)
  })
}

const tl = gsap.timeline({ onUpdate: render, defaults: { ease: 'power2.inOut', duration: 0.9 } })
```

---

## Bouncy Footer (Example 18)

Morph + elastic ease driven by scroll velocity:

```js
gsap.registerPlugin(ScrollTrigger, MorphSVGPlugin)

const down   = 'M0-0.3C0-0.3,464,156,1139,156S2278-0.3,2278-0.3V683H0V-0.3z'
const center = 'M0-0.3C0-0.3,464,0,1139,0s1139-0.3,1139-0.3V683H0V-0.3z'

ScrollTrigger.create({
  trigger: '.footer',
  start: 'top bottom',
  toggleActions: 'play pause resume reverse',
  onEnter: (self) => {
    const velocity = self.getVelocity()
    const variation = velocity / 10000

    gsap.fromTo('#bouncy-path',
      { morphSVG: down },
      {
        duration: 2,
        morphSVG: center,
        ease: `elastic.out(${1 + variation}, ${1 - variation})`,
        overwrite: true,
      }
    )
  }
})
```

---

## DrawSVGPlugin (Club)

Animates the stroke of SVG paths to look like they're drawing themselves.

```js
gsap.registerPlugin(DrawSVGPlugin)

// Draw from 0% to 100%
gsap.from('#path', { drawSVG: 0, duration: 2, ease: 'power2.inOut' })

// Draw a portion
gsap.to('#path', { drawSVG: '25% 75%', duration: 1 })

// Get current length
const length = DrawSVGPlugin.getLength('#path')

// Get current position
const [start, end] = DrawSVGPlugin.getPosition('#path')
```

### Entrance Animation Pattern

```tsx
useGSAP(() => {
  const paths = document.querySelectorAll('.icon path')
  gsap.set(paths, { drawSVG: 0 })

  gsap.to(paths, {
    drawSVG: '100%',
    duration: 1.5,
    stagger: 0.1,
    ease: 'power2.inOut',
    scrollTrigger: {
      trigger: '.icon',
      start: 'top 80%',
      once: true,
    }
  })
}, { scope: container })
```

---

## MotionPathPlugin

Move an element along an SVG path.

```js
gsap.registerPlugin(MotionPathPlugin)

// Along an SVG path element
gsap.to('.car', {
  duration: 5,
  ease: 'none',
  motionPath: {
    path: '#road',          // SVG <path> selector
    align: '#road',         // align element to path
    autoRotate: true,       // rotate element to follow path angle
    alignOrigin: [0.5, 0.5], // center of element
    start: 0,               // start at 0%
    end: 1,                 // end at 100%
  }
})

// Along an array of points
gsap.to('.element', {
  duration: 3,
  ease: 'none',
  motionPath: {
    path: [
      { x: 0,   y: 0 },
      { x: 100, y: 50 },
      { x: 200, y: 0 },
    ],
    curviness: 1.5,
  }
})
```

### Scroll-Driven MotionPath Waypoints

See [scroll.md](./scroll.md) for the full waypoints pattern.

---

## MorphSVG convertToPath

Convert shapes to paths for morphing:

```js
// Convert any SVG shape to a <path>
MorphSVGPlugin.convertToPath('#circle')
MorphSVGPlugin.convertToPath('#rect, #ellipse')

// Get raw path data
const rawPath = MorphSVGPlugin.stringToRawPath('M 0 0 L 100 100')
const str = MorphSVGPlugin.rawPathToString(rawPath)
```

---

## SVG Asset Management

When SVGs come from external sources or Codepen examples:

1. **Extract SVG code** from the reference
2. **Save to** `public/icons/` or `src/assets/svg/`
3. **Import** as React component with SVGR or as `<img src="/icon.svg">`
4. **Adapt colors** to match project design tokens

```tsx
// Using as inline SVG (required for GSAP to access paths)
import LogoSVG from '@/assets/logo.svg'

// Or inline in JSX for GSAP access
export function AnimatedLogo() {
  return (
    <svg viewBox="0 0 100 100">
      <path className="morph-path" d="M..." />
    </svg>
  )
}
```

**Important**: GSAP can only animate SVG paths that are inline in the DOM. `<img>` and CSS `background-image` SVGs are not accessible to GSAP.
