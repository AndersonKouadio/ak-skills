# Scroll Effect Patterns

## Velocity Skew on Scroll

Elements skew based on how fast the user scrolls. Smooth return to 0.

**Plugins**: ScrollTrigger, `quickSetter`

```tsx
// components/VelocitySkew.tsx
'use client'
import { useGSAP } from '@gsap/react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'

gsap.registerPlugin(ScrollTrigger)

export function VelocitySkew({ selector = '.skew-item' }) {
  useGSAP(() => {
    const proxy = { skew: 0 }
    const skewSetter = gsap.quickSetter(selector, 'skewY', 'deg')
    const clamp = gsap.utils.clamp(-20, 20)

    ScrollTrigger.create({
      onUpdate: (self) => {
        const skew = clamp(self.getVelocity() / -300)
        if (Math.abs(skew) > Math.abs(proxy.skew)) {
          proxy.skew = skew
          gsap.to(proxy, {
            skew: 0,
            duration: 0.8,
            ease: 'power3',
            overwrite: true,
            onUpdate: () => skewSetter(proxy.skew),
          })
        }
      }
    })

    gsap.set(selector, { transformOrigin: 'right center', force3D: true })
  })
}
```

---

## Scroll-Triggered Entrance (Stagger Cards)

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'
import { useRef } from 'react'

gsap.registerPlugin(ScrollTrigger)

export function CardGrid({ items }: { items: any[] }) {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    gsap.from('.card', {
      y: 60,
      opacity: 0,
      duration: 0.8,
      ease: 'power3.out',
      stagger: {
        each: 0.1,
        from: 'start',
      },
      scrollTrigger: {
        trigger: container.current,
        start: 'top 80%',
        once: true,
      }
    })
  }, { scope: container })

  return (
    <div ref={container} className="grid grid-cols-3 gap-6">
      {items.map((item, i) => (
        <div key={i} className="card">{/* ... */}</div>
      ))}
    </div>
  )
}
```

---

## ScrollTrigger Callbacks (toggleActions reference)

```js
// toggleActions: onEnter onLeave onEnterBack onLeaveBack
// Values: play, pause, resume, reverse, restart, reset, complete, none

gsap.to('.el', {
  y: -50,
  scrollTrigger: {
    trigger: '.el',
    toggleActions: 'play none none reverse',
    // Play when entering, reverse when scrolling back up
  }
})

// Common presets:
// 'play none none none'    → plays once, never reverses
// 'play pause resume none' → pause when leaving, resume on reenter
// 'play reverse play reverse' → full mirror scroll behavior
```

---

## Pinned Section with Content

```tsx
useGSAP(() => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '.pin-section',
      start: 'top top',
      end: '+=300%',
      pin: true,
      scrub: 1,
      anticipatePin: 1,
    }
  })

  tl.from('.step-1', { opacity: 0, y: 30 })
    .to('.step-1', { opacity: 0 })
    .from('.step-2', { opacity: 0, y: 30 })
    .to('.step-2', { opacity: 0 })
    .from('.step-3', { opacity: 0, y: 30 })
}, { scope: container })
```

---

## Parallax Elements

```tsx
useGSAP(() => {
  // Different scroll speeds
  gsap.to('.hero-bg', {
    yPercent: -30,
    ease: 'none',
    scrollTrigger: {
      trigger: '.hero',
      start: 'top top',
      end: 'bottom top',
      scrub: true,
    }
  })

  gsap.to('.hero-text', {
    yPercent: -60,
    ease: 'none',
    scrollTrigger: {
      trigger: '.hero',
      start: 'top top',
      end: 'bottom top',
      scrub: true,
    }
  })
})
```

---

## Before/After Image Mask Reveal (Example 25)

```tsx
useGSAP(() => {
  document.querySelectorAll<HTMLElement>('.comparison').forEach((section) => {
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: section,
        start: 'center center',
        end: () => `+=${section.offsetWidth}`,
        scrub: true,
        pin: true,
        anticipatePin: 1,
      },
      defaults: { ease: 'none' }
    })

    tl.fromTo(section.querySelector('.after-image')!,
      { xPercent: 100, x: 0 },
      { xPercent: 0 }
    )
    .fromTo(section.querySelector('.after-image img')!,
      { xPercent: -100, x: 0 },
      { xPercent: 0 },
      0
    )
  })
})
```

CSS:
```css
.comparison { position: relative; padding-bottom: 56.25%; }
.comparison-image { width: 100%; height: 100%; }
.after-image { position: absolute; overflow: hidden; top: 0; transform: translate(100%, 0); }
.after-image img { transform: translate(-100%, 0); }
.comparison-image img { width: 100%; height: 100%; position: absolute; top: 0; }
```

---

## Lateral Progress Indicator (Example 26)

Pinned section with list + image switcher tied to scroll progress:

```tsx
useGSAP(() => {
  const listItems = gsap.utils.toArray<HTMLElement>('li', '.list')
  const slides = gsap.utils.toArray<HTMLElement>('.slide')
  const fill = document.querySelector<HTMLElement>('.fill')

  if (fill) gsap.set(fill, { scaleY: 1 / listItems.length, transformOrigin: 'top left' })

  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '.pin-section',
      start: 'top top',
      end: `+=${listItems.length * 50}%`,
      pin: true,
      scrub: true,
    }
  })

  listItems.forEach((item, i) => {
    const prev = listItems[i - 1]
    if (prev) {
      tl.set(item, { color: '#0ae448' }, 0.5 * i)
        .to(slides[i], { autoAlpha: 1, duration: 0.2 }, '<')
        .set(prev, { color: '#fffce1' }, '<')
        .to(slides[i - 1], { autoAlpha: 0, duration: 0.2 }, '<')
    } else {
      gsap.set(item, { color: '#0ae448' })
      gsap.set(slides[i], { autoAlpha: 1 })
    }
  })

  if (fill) {
    tl.to(fill, { scaleY: 1, transformOrigin: 'top left', ease: 'none', duration: tl.duration() }, 0)
      .to({}, {}) // pause at end
  }
}, { scope: container })
```
