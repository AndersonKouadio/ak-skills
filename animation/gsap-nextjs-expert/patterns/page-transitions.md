# Page Transition Patterns

## Infinite Looped Section Transitions (Example 28)

Fixed panels with scale/fade, infinite scroll wrapping:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'

gsap.registerPlugin(ScrollTrigger)

export function InfiniteLoop() {
  useGSAP(() => {
    const sections = gsap.utils.toArray<HTMLElement>('section')
    let currentSection = sections[0]

    gsap.defaults({ overwrite: 'auto', duration: 0.3 })
    gsap.set('body', { height: `${sections.length * 100}%` })

    sections.forEach((section, i) => {
      ScrollTrigger.create({
        start: () => (i - 0.5) * window.innerHeight,
        end: () => (i + 0.5) * window.innerHeight,
        onToggle: (self) => self.isActive && setSection(section),
      })
    })

    function setSection(newSection: HTMLElement) {
      if (newSection !== currentSection) {
        gsap.to(currentSection, { scale: 0.8, autoAlpha: 0 })
        gsap.to(newSection, { scale: 1, autoAlpha: 1 })
        currentSection = newSection
      }
    }

    // Wrap around at top and bottom
    ScrollTrigger.create({
      start: 1,
      end: () => ScrollTrigger.maxScroll(window) - 1,
      onLeaveBack: (self) => self.scroll(ScrollTrigger.maxScroll(window) - 2),
      onLeave: (self) => self.scroll(2),
    }).scroll(2)
  })

  return (
    <>
      {/* sections must be position:fixed, full viewport */}
    </>
  )
}
```

CSS:
```css
section {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0; left: 0;
}
section:not(:first-child) {
  opacity: 0;
  visibility: hidden;
  transform: scale(0.8);
}
```

---

## Layered Pinned Panels (Example 15)

Panels stack on top of each other with pinning + snap:

```tsx
useGSAP(() => {
  const panels = gsap.utils.toArray<HTMLElement>('.panel')
  // Clone first panel to the end for seamless loop
  const copy = panels[0].cloneNode(true) as HTMLElement
  panels[0].parentNode!.appendChild(copy)

  panels.forEach((panel) => {
    ScrollTrigger.create({
      trigger: panel,
      start: 'top top',
      pin: true,
      pinSpacing: false,
    })
  })

  let maxScroll: number
  const pageScrollTrigger = ScrollTrigger.create({
    snap(value) {
      const snapped = gsap.utils.snap(1 / panels.length, value)
      if (snapped <= 0) return 1.05 / maxScroll
      if (snapped >= 1) return maxScroll / (maxScroll + 1.05)
      return snapped
    }
  })

  const updateMax = () => { maxScroll = ScrollTrigger.maxScroll(window) - 1 }
  updateMax()
  window.addEventListener('resize', updateMax)

  window.addEventListener('scroll', (e) => {
    const scroll = pageScrollTrigger.scroll()
    if (scroll > maxScroll) { pageScrollTrigger.scroll(1); e.preventDefault() }
    else if (scroll < 1) { pageScrollTrigger.scroll(maxScroll - 1); e.preventDefault() }
  }, { passive: false })
})
```

---

## Route Transition (Next.js App Router)

Overlay morphs in/out when navigating between pages:

```tsx
// components/PageTransition.tsx
'use client'
import { usePathname } from 'next/navigation'
import { useEffect, useRef } from 'react'
import { useGSAP } from '@gsap/react'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import gsap from 'gsap'

gsap.registerPlugin(MorphSVGPlugin)

const FLAT   = 'M 0 100 V 100 Q 50 100 100 100 V 100 z'
const CURVED = 'M 0 100 V 50 Q 50 0 100 50 V 100 z'
const FULL   = 'M 0 100 V 0 Q 50 0 100 0 V 100 z'

export function PageTransition() {
  const pathname = usePathname()
  const pathRef = useRef<SVGPathElement>(null)

  useEffect(() => {
    const el = pathRef.current!
    const tl = gsap.timeline()

    // Sweep in
    tl.set(el, { morphSVG: FLAT })
      .to(el, { morphSVG: CURVED, duration: 0.35, ease: 'power2.in' })
      .to(el, { morphSVG: FULL,   duration: 0.35, ease: 'power2.out' })
      // Hold briefly, then sweep out
      .to(el, { morphSVG: CURVED, duration: 0.3,  ease: 'power2.in',  delay: 0.1 })
      .to(el, { morphSVG: FLAT,   duration: 0.35, ease: 'power2.out' })
  }, [pathname])

  return (
    <svg
      style={{ position: 'fixed', inset: 0, width: '100%', height: '100%', zIndex: 9999, pointerEvents: 'none' }}
      viewBox="0 0 100 100"
      preserveAspectRatio="xMidYMin slice"
    >
      <path ref={pathRef} fill="var(--brand-bg)" d={FLAT} />
    </svg>
  )
}
```

---

## Smooth Section Switching with Observer

Full-page sections, scroll/swipe to navigate — no scrollbar:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { Observer } from 'gsap/Observer'
import gsap from 'gsap'

gsap.registerPlugin(Observer)

const SECTIONS = ['#hero', '#about', '#work', '#contact']

export function FullPageSections() {
  useGSAP(() => {
    let current = 0
    let animating = false
    const sections = SECTIONS.map(s => document.querySelector<HTMLElement>(s)!)

    gsap.set(sections, { yPercent: (i) => i === 0 ? 0 : 100 })

    function goTo(index: number) {
      if (index === current || index < 0 || index >= sections.length || animating) return
      animating = true

      const dir = index > current ? 1 : -1
      const tl = gsap.timeline({
        defaults: { duration: 1, ease: 'power3.inOut' },
        onComplete: () => { animating = false }
      })

      tl.to(sections[current], { yPercent: -100 * dir })
        .fromTo(sections[index], { yPercent: 100 * dir }, { yPercent: 0 }, '<')

      current = index
    }

    Observer.create({
      type: 'wheel,touch,pointer',
      wheelSpeed: -1,
      onDown: () => goTo(current + 1),
      onUp: () => goTo(current - 1),
      tolerance: 10,
      preventDefault: true,
    })
  })

  return null
}
```
