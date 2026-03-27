# Text Animation Patterns

## Hero Entrance Sequence

Full hero: heading + subtitle + CTA button, staggered in:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { SplitText } from 'gsap/SplitText'
import gsap from 'gsap'
import { useRef } from 'react'

gsap.registerPlugin(SplitText)

export function HeroEntrance() {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    document.fonts.ready.then(() => {
      const tl = gsap.timeline({ defaults: { ease: 'power3.out' } })

      // Split the heading
      const split = SplitText.create('.hero-heading', { type: 'lines', mask: 'lines' })

      tl.from(split.lines, { yPercent: 110, duration: 1, stagger: 0.12 })
        .from('.hero-subtitle', { opacity: 0, y: 20, duration: 0.8 }, '-=0.4')
        .from('.hero-cta', { opacity: 0, y: 20, scale: 0.9, duration: 0.6 }, '-=0.5')
    })
  }, { scope: container })

  return (
    <div ref={container} className="hero">
      <h1 className="hero-heading">Your bold headline here</h1>
      <p className="hero-subtitle">Supporting text that explains the value.</p>
      <button className="hero-cta">Get started</button>
    </div>
  )
}
```

---

## Rolling Tube Text (Example 9)

3D cylindrical text animation — chars rotate on X axis in 3D space:

See [text.md](../text.md) for full implementation.

Key settings:
- `perspective: 700` on parent
- `transformStyle: 'preserve-3d'`
- `transformOrigin: '50% 50% ${depth}'` where depth = `-width / 8`
- `rotationX: -90 → 90` with stagger

---

## Word-by-Word Reveal on Scroll

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const paragraphs = gsap.utils.toArray<HTMLElement>('.reveal-text')

    paragraphs.forEach((p) => {
      SplitText.create(p, {
        type: 'words',
        mask: 'words',
        autoSplit: true,
        onSplit: (instance) => {
          return gsap.from(instance.words, {
            opacity: 0,
            y: 15,
            stagger: 0.035,
            duration: 0.6,
            ease: 'power2.out',
            scrollTrigger: {
              trigger: p,
              start: 'top 85%',
              once: true,
            }
          })
        }
      })
    })
  })
}, { scope: container })
```

---

## Counter Animation

```tsx
useGSAP(() => {
  document.querySelectorAll<HTMLElement>('.counter').forEach((el) => {
    const target = parseInt(el.dataset.target || '0')
    gsap.from({ val: 0 }, {
      val: target,
      duration: 2,
      ease: 'power2.out',
      roundProps: 'val',
      onUpdate() {
        el.textContent = (this as any).targets()[0].val.toLocaleString()
      },
      scrollTrigger: {
        trigger: el,
        start: 'top 80%',
        once: true,
      }
    })
  })
}, { scope: container })
```

---

## Staggered Word Blur-in

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const split = SplitText.create('.blur-text', { type: 'words' })

    gsap.from(split.words, {
      opacity: 0,
      filter: 'blur(12px)',
      stagger: 0.05,
      duration: 0.8,
      ease: 'power2.out',
      scrollTrigger: { trigger: '.blur-text', start: 'top 80%', once: true }
    })
  })
}, { scope: container })
```

---

## Typewriter Effect (TextPlugin)

```tsx
import { TextPlugin } from 'gsap/TextPlugin'
gsap.registerPlugin(TextPlugin)

useGSAP(() => {
  const phrases = ['Design', 'Animate', 'Ship']
  let i = 0

  const tl = gsap.timeline({ repeat: -1 })

  phrases.forEach((phrase) => {
    tl.to('.typewriter', { text: phrase, duration: 0.8, ease: 'none' })
      .to('.typewriter', { text: '', duration: 0.4, ease: 'none', delay: 1.5 })
  })
}, { scope: container })
```

---

## Headline with Gradient Reveal

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const split = SplitText.create('.gradient-headline', { type: 'chars' })

    gsap.from(split.chars, {
      opacity: 0,
      y: 50,
      rotationX: -90,
      transformOrigin: '0% 50% -50px',
      stagger: { each: 0.04, from: 'start' },
      duration: 0.8,
      ease: 'back.out(1.7)',
    })
  })
}, { scope: container })
```

---

## Responsive Line Splits (Example 30)

The definitive pattern for line-based text animations that survive resize:

See [text.md](../text.md) — uses `autoSplit: true` + return animation from `onSplit` callback.

Critical rules:
1. Return the animation from `onSplit` — GSAP uses it for cleanup on re-split
2. Set `autoSplit: true` — triggers `onSplit` again on resize
3. Use `mask: 'lines'` — prevents text peeking above/below during animation
4. Wait for `document.fonts.ready` — prevents wrong line breaks
