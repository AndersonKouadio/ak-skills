# Scroll Animations

## ScrollTrigger — Core Pattern

```js
gsap.registerPlugin(ScrollTrigger)

gsap.from('.element', {
  y: 60,
  opacity: 0,
  duration: 1,
  scrollTrigger: {
    trigger: '.element',
    start: 'top 80%',         // [element edge] [viewport edge]
    end: 'bottom 20%',
    toggleActions: 'play none none reverse', // onEnter onLeave onEnterBack onLeaveBack
    // possible values: play, pause, resume, reverse, restart, reset, complete, none
    scrub: true,              // tie animation to scroll position (true = smooth, number = lag)
    pin: true,                // pin element during scroll
    pinSpacing: false,        // don't add spacing after pin
    markers: true,            // DEV ONLY — show start/end markers
    anticipatePin: 1,         // reduce pin jump
    once: true,               // only trigger once
    invalidateOnRefresh: true,// recalculate on resize
  }
})
```

### Start/End Position Strings

```
'top top'       → element top hits viewport top
'top center'    → element top hits viewport center
'top 80%'       → element top hits 80% from viewport top
'center center' → element center hits viewport center
'bottom bottom' → element bottom hits viewport bottom
'+=300'         → 300px after the start
'+=100%'        → 100% of viewport height after start
'clamp(top center)' → clamp so it can't trigger before page loads
```

---

## Scrub — Scroll-Driven Animation

```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: '.section',
    start: 'top top',
    end: '+=1000',
    scrub: 1,   // 1 second lag for smoothness (0.5–2 recommended)
    pin: true,
  }
})
tl.from('.title', { y: 100, opacity: 0 })
  .from('.img', { scale: 0.8 }, '<')
```

---

## Pin — Freeze During Scroll

```js
// Pin a section for 500px of scrolling
ScrollTrigger.create({
  trigger: '.hero',
  start: 'top top',
  end: '+=500',
  pin: true,
  pinSpacing: true,  // default: adds scroll space after
})

// Pin without spacing (stacking panels)
ScrollTrigger.create({
  trigger: '.panel',
  start: 'top top',
  pin: true,
  pinSpacing: false,
})
```

---

## Velocity Effects

### Scroll velocity → skew (Example 4)
```js
let proxy = { skew: 0 }
const skewSetter = gsap.quickSetter('.skewElem', 'skewY', 'deg')
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
        onUpdate: () => skewSetter(proxy.skew)
      })
    }
  }
})
gsap.set('.skewElem', { transformOrigin: 'right center', force3D: true })
```

### Scroll velocity → WebGL/shader
```js
const velocityProxy = { v: 0, s: 0 }
const clamp = gsap.utils.clamp(-2000, 2000)

ScrollTrigger.create({
  start: 0,
  end: () => document.documentElement.scrollHeight - window.innerHeight,
  onUpdate(self) {
    const raw = clamp(self.getVelocity())
    const norm = raw / 1000
    const strength = Math.min(1, Math.abs(norm))
    if (Math.abs(strength) > Math.abs(velocityProxy.s)) {
      velocityProxy.v = norm
      velocityProxy.s = strength
      gsap.to(velocityProxy, { v: 0, s: 0, duration: 0.8, ease: 'sine.inOut', overwrite: true })
    }
  }
})
```

---

## Snap

```js
ScrollTrigger.create({
  trigger: '.container',
  start: 'top top',
  end: '+=3000',
  snap: {
    snapTo: 1 / (sections.length - 1),  // fraction of total scroll
    duration: { min: 0.2, max: 0.5 },
    ease: 'power1.inOut',
    inertia: false,
  }
})

// Snap to array of progress values
snap: { snapTo: [0, 0.25, 0.5, 0.75, 1] }

// Snap with custom function
snap: {
  snapTo: (value) => gsap.utils.snap(0.1, value)
}
```

---

## containerAnimation — Nested ScrollTrigger

Used when elements are inside a horizontally-animated container:

```js
// Parent horizontal scroll
const scrollTween = gsap.to('.strip', {
  xPercent: -100,
  ease: 'none',
  scrollTrigger: {
    trigger: '.wrapper',
    pin: true,
    scrub: true,
    end: '+=5000',
  }
})

// Child animations triggered within that horizontal scroll
gsap.from('.card', {
  scale: 0.8,
  opacity: 0,
  scrollTrigger: {
    trigger: '.card',
    containerAnimation: scrollTween,  // ← key
    start: 'left 100%',
    end: 'left 50%',
    scrub: 1,
  }
})
```

---

## ScrollTrigger Callbacks

```js
ScrollTrigger.create({
  trigger: '.el',
  onEnter: (self) => console.log('entered'),
  onLeave: (self) => console.log('left'),
  onEnterBack: (self) => console.log('entered back'),
  onLeaveBack: (self) => console.log('left back'),
  onUpdate: (self) => {
    // self.progress — 0 to 1
    // self.direction — 1 (down) or -1 (up)
    // self.velocity — scroll velocity
    // self.getVelocity() — current velocity
  },
  onRefresh: (self) => {},
  onScrubComplete: (self) => {},
})
```

---

## Batch — Animate Many Elements

```js
// Animate elements as they enter the viewport, in batches
ScrollTrigger.batch('.card', {
  onEnter: (elements) => {
    gsap.from(elements, {
      y: 60,
      opacity: 0,
      stagger: 0.1,
      overwrite: true,
    })
  },
  onLeave: (elements) => {
    gsap.set(elements, { opacity: 0, y: -60, overwrite: true })
  },
  onEnterBack: (elements) => {
    gsap.to(elements, { opacity: 1, y: 0, stagger: 0.1, overwrite: true })
  },
  batchMax: 3,       // max elements per batch
  start: 'top 85%',
})
```

---

## ScrollSmoother (Club Plugin)

Requires `#smooth-wrapper` and `#smooth-content` in the DOM.
**Must be created BEFORE any ScrollTrigger instances.**

```js
gsap.registerPlugin(ScrollSmoother, ScrollTrigger)

const smoother = ScrollSmoother.create({
  wrapper: '#smooth-wrapper',
  content: '#smooth-content',
  smooth: 2,              // smoothing lag (0 = none)
  effects: true,          // enable data-speed / data-lag attributes
  normalizeScroll: true,  // fix mobile scroll behavior
  ignoreMobileResize: true,
})

// Programmatic scroll
smoother.scrollTo('.target', true, 'center center')
smoother.scrollTop(500)

// Get current scroll position
smoother.scrollTop()

// Pause/resume
smoother.paused(true)
```

### HTML Attributes for Effects

```html
<!-- Parallax speed (1 = normal, 0.5 = slower, 1.5 = faster) -->
<div data-speed="0.7">Slow parallax</div>
<div data-speed="clamp(0.5)">Clamped (won't go outside viewport)</div>

<!-- Lag (delayed catch-up) -->
<div data-lag="0.5">Laggy element</div>
```

---

## Horizontal Gallery (Example 27)

```js
const smoother = ScrollSmoother.create({
  wrapper: '#smooth-wrapper',
  content: '#smooth-content',
  smooth: 2,
  normalizeScroll: true,
})

const sec = document.querySelector('.horiz-gallery-wrapper')
const pinWrap = sec.querySelector('.horiz-gallery-strip')

let pinWrapWidth = pinWrap.scrollWidth
let horizontalScrollLength = pinWrapWidth - window.innerWidth

gsap.to(pinWrap, {
  scrollTrigger: {
    scrub: true,
    trigger: sec,
    pin: sec,
    start: 'center center',
    end: () => `+=${pinWrapWidth}`,
    invalidateOnRefresh: true,
  },
  x: () => -horizontalScrollLength,
  ease: 'none',
})
```

---

## Infinite Pinned Panels (Example 15)

```js
const panels = gsap.utils.toArray('.panel')
const copy = panels[0].cloneNode(true)
panels[0].parentNode.appendChild(copy) // seamless loop

panels.forEach((panel) => {
  ScrollTrigger.create({
    trigger: panel,
    start: 'top top',
    pin: true,
    pinSpacing: false,
  })
})

let maxScroll
const pageScrollTrigger = ScrollTrigger.create({
  snap(value) {
    const snapped = gsap.utils.snap(1 / panels.length, value)
    if (snapped <= 0) return 1.05 / maxScroll
    if (snapped >= 1) return maxScroll / (maxScroll + 1.05)
    return snapped
  }
})

function onResize() { maxScroll = ScrollTrigger.maxScroll(window) - 1 }
onResize()
window.addEventListener('resize', onResize)

window.addEventListener('scroll', (e) => {
  const scroll = pageScrollTrigger.scroll()
  if (scroll > maxScroll) { pageScrollTrigger.scroll(1); e.preventDefault() }
  else if (scroll < 1) { pageScrollTrigger.scroll(maxScroll - 1); e.preventDefault() }
}, { passive: false })
```

---

## ScrollTrigger Refresh & Invalidate

```js
// Force recalculate all ScrollTriggers (call after DOM changes)
ScrollTrigger.refresh()

// Invalidate + restart a specific tween (recalculate starting values)
tween.invalidate().restart()

// Kill all ScrollTriggers
ScrollTrigger.killAll()

// Get all
ScrollTrigger.getAll()

// Get by ID
ScrollTrigger.getById('myId')

// Save styles before ScrollTrigger modifies them (for cleanup)
ScrollTrigger.saveStyles('.el, .other')
```

---

## MotionPath Waypoints on Scroll (Example 14)

```js
gsap.registerPlugin(ScrollTrigger, MotionPathPlugin)

let ctx

function createTimeline() {
  ctx?.revert()
  ctx = gsap.context(() => {
    const box = document.querySelector('.box')
    const boxRect = box.getBoundingClientRect()
    const containers = gsap.utils.toArray('.container:not(.initial)')

    const points = containers.map((c) => {
      const r = c.querySelector('.marker').getBoundingClientRect()
      return {
        x: r.left + r.width / 2 - (boxRect.left + boxRect.width / 2),
        y: r.top + r.height / 2 - (boxRect.top + boxRect.height / 2),
      }
    })

    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: '.container.initial',
        start: 'clamp(top center)',
        endTrigger: '.final',
        end: 'clamp(top center)',
        scrub: 1,
      }
    })

    tl.to('.box', {
      duration: 1,
      ease: 'none',
      motionPath: { path: points, curviness: 1.5 }
    })
  })
}

createTimeline()
window.addEventListener('resize', createTimeline)
```

---

## Image Sequence on Scroll (Example 17)

```js
function imageSequence(config) {
  const playhead = { frame: 0 }
  const canvas = gsap.utils.toArray(config.canvas)[0]
  const ctx = canvas.getContext('2d')
  let curFrame = -1

  const images = config.urls.map((url, i) => {
    const img = new Image()
    img.src = url
    if (i === 0) img.onload = updateImage
    return img
  })

  function updateImage() {
    const frame = Math.round(playhead.frame)
    if (frame !== curFrame) {
      config.clear && ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.drawImage(images[frame], 0, 0)
      curFrame = frame
      config.onUpdate?.call(this, frame, images[frame])
    }
  }

  return gsap.to(playhead, {
    frame: images.length - 1,
    ease: 'none',
    onUpdate: updateImage,
    duration: images.length / (config.fps || 30),
    paused: !!config.paused,
    scrollTrigger: config.scrollTrigger,
  })
}

// Usage
imageSequence({
  urls: frameUrls,
  canvas: '#canvas',
  scrollTrigger: { start: 0, end: 'max', scrub: true }
})
```
