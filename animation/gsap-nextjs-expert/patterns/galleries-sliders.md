# Gallery & Slider Patterns

## Infinite Card Slider (Example 6)

Scroll-driven infinite loop with drag support. Uses `buildSeamlessLoop` helper.

**Plugins**: ScrollTrigger, Draggable

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { Draggable } from 'gsap/Draggable'
import gsap from 'gsap'

gsap.registerPlugin(ScrollTrigger, Draggable)

export function InfiniteCardSlider({ cards }: { cards: string[] }) {
  useGSAP(() => {
    const spacing = 0.1
    const snapTime = gsap.utils.snap(spacing)
    const cardEls = gsap.utils.toArray<HTMLElement>('.card-item')

    gsap.set(cardEls, { xPercent: 400, opacity: 0, scale: 0 })

    const animateFunc = (el: HTMLElement) => {
      const tl = gsap.timeline()
      tl.fromTo(el,
        { scale: 0, opacity: 0 },
        { scale: 1, opacity: 1, zIndex: 100, duration: 0.5, yoyo: true, repeat: 1, ease: 'power1.in', immediateRender: false }
      )
      .fromTo(el,
        { xPercent: 400 },
        { xPercent: -400, duration: 1, ease: 'none', immediateRender: false },
        0
      )
      return tl
    }

    const seamlessLoop = buildSeamlessLoop(cardEls, spacing, animateFunc)
    const wrapTime = gsap.utils.wrap(0, seamlessLoop.duration())
    let iteration = 0

    const playhead = { offset: 0 }
    const scrub = gsap.to(playhead, {
      offset: 0,
      onUpdate() { seamlessLoop.time(wrapTime(playhead.offset)) },
      duration: 0.5,
      ease: 'power3',
      paused: true,
    })

    const trigger = ScrollTrigger.create({
      start: 0,
      onUpdate(self) {
        const scroll = self.scroll()
        if (scroll > self.end - 1) {
          wrap(1, 2)
        } else if (scroll < 1 && self.direction < 0) {
          wrap(-1, self.end - 2)
        } else {
          scrub.vars.offset = (iteration + self.progress) * seamlessLoop.duration()
          scrub.invalidate().restart()
        }
      },
      end: '+=3000',
      pin: '.gallery',
    })

    const progressToScroll = (p: number) =>
      gsap.utils.clamp(1, trigger.end - 1, gsap.utils.wrap(0, 1, p) * trigger.end)

    const wrap = (delta: number, scrollTo: number) => {
      iteration += delta
      trigger.scroll(scrollTo)
      trigger.update()
    }

    ScrollTrigger.addEventListener('scrollEnd', () => scrollToOffset(scrub.vars.offset))

    function scrollToOffset(offset: number) {
      const snapped = snapTime(offset)
      const progress = (snapped - seamlessLoop.duration() * iteration) / seamlessLoop.duration()
      const scroll = progressToScroll(progress)
      if (progress >= 1 || progress < 0) return wrap(Math.floor(progress), scroll)
      trigger.scroll(scroll)
    }

    document.querySelector('.next')?.addEventListener('click', () => scrollToOffset(scrub.vars.offset + spacing))
    document.querySelector('.prev')?.addEventListener('click', () => scrollToOffset(scrub.vars.offset - spacing))

    Draggable.create('.drag-proxy', {
      type: 'x',
      trigger: '.cards',
      onPress() { (this as any).startOffset = scrub.vars.offset },
      onDrag() {
        scrub.vars.offset = (this as any).startOffset + ((this as any).startX - (this as any).x) * 0.001
        scrub.invalidate().restart()
      },
      onDragEnd() { scrollToOffset(scrub.vars.offset) }
    })
  })

  return (
    <div className="gallery">
      <ul className="cards">
        {cards.map((src, i) => (
          <li key={i} className="card-item" style={{ backgroundImage: `url(${src})` }} />
        ))}
      </ul>
      <div className="actions">
        <button className="prev">Prev</button>
        <button className="next">Next</button>
      </div>
      <div className="drag-proxy" style={{ visibility: 'hidden', position: 'absolute' }} />
    </div>
  )
}

// Helper: builds seamless looping timeline
function buildSeamlessLoop(items: HTMLElement[], spacing: number, animateFunc: (el: HTMLElement) => gsap.core.Timeline) {
  const overlap = Math.ceil(1 / spacing)
  const startTime = items.length * spacing + 0.5
  const loopTime = (items.length + overlap) * spacing + 1
  const rawSequence = gsap.timeline({ paused: true })
  const seamlessLoop = gsap.timeline({
    paused: true,
    repeat: -1,
    onRepeat() {
      if ((this as any)._time === (this as any)._dur) {
        (this as any)._tTime += (this as any)._dur - 0.01
      }
    }
  })
  const l = items.length + overlap * 2

  for (let i = 0; i < l; i++) {
    const index = i % items.length
    const time = i * spacing
    rawSequence.add(animateFunc(items[index]), time)
    if (i <= items.length) seamlessLoop.add('label' + i, time)
  }

  rawSequence.time(startTime)
  seamlessLoop
    .to(rawSequence, { time: loopTime, duration: loopTime - startTime, ease: 'none' })
    .fromTo(rawSequence,
      { time: overlap * spacing + 1 },
      { time: startTime, duration: startTime - (overlap * spacing + 1), immediateRender: false, ease: 'none' }
    )

  return seamlessLoop
}
```

---

## Horizontal Gallery with ScrollSmoother (Example 27)

```tsx
useGSAP(() => {
  const horizontalSections = gsap.utils.toArray<HTMLElement>('.horiz-gallery-wrapper')

  horizontalSections.forEach((sec) => {
    const strip = sec.querySelector<HTMLElement>('.horiz-gallery-strip')!

    let pinWrapWidth = strip.scrollWidth
    let scrollLength = pinWrapWidth - window.innerWidth

    const refresh = () => {
      pinWrapWidth = strip.scrollWidth
      scrollLength = pinWrapWidth - window.innerWidth
    }

    refresh()

    gsap.to(strip, {
      scrollTrigger: {
        scrub: true,
        trigger: sec,
        pin: sec,
        start: 'center center',
        end: () => `+=${pinWrapWidth}`,
        invalidateOnRefresh: true,
      },
      x: () => -scrollLength,
      ease: 'none',
    })

    ScrollTrigger.addEventListener('refreshInit', refresh)
  })
})
```

---

## Auto-scrolling Marquee / Ticker

```tsx
useGSAP(() => {
  const items = gsap.utils.toArray<HTMLElement>('.marquee-item')
  const totalWidth = items.reduce((w, el) => w + el.offsetWidth, 0)

  gsap.to('.marquee-track', {
    x: -totalWidth / 2,  // move half (assuming items are duplicated)
    duration: 20,
    ease: 'none',
    repeat: -1,
  })
}, { scope: container })
```

---

## Pinned Panels with Overscroll (Example 16)

Each panel can have scrollable inner content before the next panel slides in:

```tsx
useGSAP(() => {
  const panels = gsap.utils.toArray<HTMLElement>('.section')
  panels.pop() // remove last panel (no transition needed)

  panels.forEach((panel) => {
    const inner = panel.querySelector<HTMLElement>('.section-inner')!
    const panelHeight = inner.offsetHeight
    const windowH = window.innerHeight
    const diff = panelHeight - windowH
    const fakeScrollRatio = diff > 0 ? diff / (diff + windowH) : 0

    if (fakeScrollRatio) {
      panel.style.marginBottom = `${panelHeight * fakeScrollRatio}px`
    }

    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: panel,
        start: 'bottom bottom',
        end: () => fakeScrollRatio ? `+=${inner.offsetHeight}` : 'bottom top',
        pinSpacing: false,
        pin: true,
        scrub: true,
      }
    })

    if (fakeScrollRatio) {
      tl.to(inner, { yPercent: -100, y: windowH, duration: 1 / (1 - fakeScrollRatio) - 1, ease: 'none' })
    }
    tl.fromTo(panel, { scale: 1, opacity: 1 }, { scale: 0.7, opacity: 0.5, duration: 0.9 })
      .to(panel, { opacity: 0, duration: 0.1 })
  })
})
```
