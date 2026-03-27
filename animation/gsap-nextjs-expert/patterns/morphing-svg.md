# SVG Morphing Patterns

## Curve Swipe / Page Transition (Example 11)

Toggle between two path shapes on click:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import gsap from 'gsap'

gsap.registerPlugin(MorphSVGPlugin)

export function CurveTransition({ isOpen }: { isOpen: boolean }) {
  useGSAP(() => {
    const path = document.querySelector<SVGPathElement>('.transition-path')!
    const collapsed = 'M 0 100 V 100 Q 50 100 100 100 V 100 z'
    const midCurve   = 'M 0 100 V 50 Q 50 0 100 50 V 100 z'
    const fullScreen = 'M 0 100 V 0 Q 50 0 100 0 V 100 z'

    const tl = gsap.timeline()
    tl.to(path, { morphSVG: midCurve,   ease: 'power2.in' })
      .to(path, { morphSVG: fullScreen, ease: 'power2.out' })
      .reverse()

    document.body.addEventListener('click', () => {
      tl.reversed(!tl.reversed())
    })
  })

  return (
    <svg
      className="transition"
      viewBox="0 0 100 100"
      preserveAspectRatio="xMidYMin slice"
      style={{ position: 'fixed', inset: 0, width: '100%', height: '100%', pointerEvents: 'none' }}
    >
      <path
        className="transition-path"
        fill="currentColor"
        d="M 0 100 V 100 Q 50 100 100 100 V 100 z"
      />
    </svg>
  )
}
```

---

## Multi-path Wave Overlay (Example 12)

Click to reveal/hide with staggered wave effect:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'
import { useRef } from 'react'

export function WaveOverlay() {
  const tlRef = useRef<gsap.core.Timeline>()

  useGSAP(() => {
    const paths = document.querySelectorAll<SVGPathElement>('.wave-path')
    const NUM_POINTS = 10
    const NUM_PATHS = paths.length
    const DELAY_MAX = 0.3
    const DELAY_PER_PATH = 0.25

    const allPoints: number[][] = Array.from({ length: NUM_PATHS }, () =>
      Array(NUM_POINTS).fill(100)
    )

    let isOpen = false

    tlRef.current = gsap.timeline({
      onUpdate: render,
      defaults: { ease: 'power2.inOut', duration: 0.9 }
    })

    function toggle() {
      const tl = tlRef.current!
      tl.progress(0).clear()

      const pointsDelay = Array.from({ length: NUM_POINTS }, () => Math.random() * DELAY_MAX)

      for (let i = 0; i < NUM_PATHS; i++) {
        const pathDelay = DELAY_PER_PATH * (isOpen ? i : NUM_PATHS - i - 1)
        for (let j = 0; j < NUM_POINTS; j++) {
          tl.to(allPoints[i], { [j]: 0 }, pointsDelay[j] + pathDelay)
        }
      }
    }

    function render() {
      paths.forEach((path, i) => {
        const points = allPoints[i]
        let d = isOpen ? `M 0 0 V ${points[0]} C` : `M 0 ${points[0]} C`

        for (let j = 0; j < NUM_POINTS - 1; j++) {
          const p = (j + 1) / (NUM_POINTS - 1) * 100
          const cp = p - (1 / (NUM_POINTS - 1) * 100) / 2
          d += ` ${cp} ${points[j]} ${cp} ${points[j + 1]} ${p} ${points[j + 1]}`
        }
        d += isOpen ? ' V 100 H 0' : ' V 0 H 0'
        path.setAttribute('d', d)
      })
    }

    document.querySelector('.wave-overlay')?.addEventListener('click', () => {
      if (!tlRef.current!.isActive()) {
        isOpen = !isOpen
        toggle()
      }
    })

    toggle()
  })

  return (
    <svg
      className="wave-overlay"
      viewBox="0 0 100 100"
      preserveAspectRatio="none"
      style={{ width: '100%', height: '100%', position: 'fixed', top: 0, left: 0, cursor: 'pointer' }}
    >
      <path className="wave-path" fill="var(--color-2)" />
      <path className="wave-path" fill="var(--color-1)" />
    </svg>
  )
}
```

---

## Elastic Footer (Example 18)

SVG footer that bounces when scrolled into view, with velocity-based elastic intensity:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'

gsap.registerPlugin(MorphSVGPlugin, ScrollTrigger)

// The two path states — design these in Figma/Inkscape then paste here
const DOWN   = 'M0-0.3C0-0.3,464,156,1139,156S2278-0.3,2278-0.3V683H0V-0.3z'
const CENTER = 'M0-0.3C0-0.3,464,0,1139,0s1139-0.3,1139-0.3V683H0V-0.3z'

export function ElasticFooter() {
  useGSAP(() => {
    ScrollTrigger.create({
      trigger: '.footer',
      start: 'top bottom',
      toggleActions: 'play pause resume reverse',
      onEnter: (self) => {
        const velocity = self.getVelocity()
        const v = velocity / 10000

        gsap.fromTo('#footer-path',
          { morphSVG: DOWN },
          {
            duration: 2,
            morphSVG: CENTER,
            ease: `elastic.out(${1 + v}, ${1 - v})`,
            overwrite: true,
          }
        )
      }
    })
  })

  return (
    <footer className="footer">
      <svg
        id="footer-svg"
        viewBox="0 0 2278 683"
        preserveAspectRatio="none"
        style={{ width: '100%', height: '100%', display: 'block' }}
      >
        <path id="footer-path" fill="var(--brand-color)" d={DOWN} />
      </svg>
    </footer>
  )
}
```

---

## SVG Path Tips

- `preserveAspectRatio="none"` — stretch to fill container (for fullscreen transitions)
- `preserveAspectRatio="xMidYMin slice"` — cover-fill from top
- `vector-effect="non-scaling-stroke"` — keep stroke width consistent when SVG scales
- For morphing, paths should ideally have the same number of points — MorphSVG handles mismatches but smoother with matching counts
- Always convert shapes (rect, circle, ellipse) to paths before morphing: `MorphSVGPlugin.convertToPath('#shape')`
