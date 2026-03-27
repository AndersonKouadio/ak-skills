# Core GSAP API Reference

## Tweens — gsap.to / from / fromTo / set

```js
// Animate TO final values
gsap.to('.box', { x: 200, opacity: 1, duration: 1, ease: 'power3.out' })

// Animate FROM initial values (element starts at these, animates to current)
gsap.from('.box', { y: 60, opacity: 0, duration: 0.8 })

// Animate FROM → TO (explicit start and end)
gsap.fromTo('.box', { scale: 0 }, { scale: 1, ease: 'elastic.out(1, 0.3)' })

// Instantly set properties (no animation)
gsap.set('.box', { transformOrigin: '50% 50%', force3D: true })
```

---

## Timeline — Sequencing

```js
const tl = gsap.timeline({
  defaults: { ease: 'power3.out', duration: 0.8 },
  onComplete: () => console.log('done'),
  paused: true,       // don't autoplay
  repeat: -1,         // loop forever
  yoyo: true,         // reverse on repeat
  delay: 0.5,
})

tl.from('.title', { y: 60, opacity: 0 })
  .from('.subtitle', { y: 40, opacity: 0 }, '-=0.4')  // overlap 0.4s
  .from('.cta', { scale: 0.8, opacity: 0 }, '<0.2')   // 0.2s after previous starts
  .to('.bg', { scale: 1.1 }, 0)                        // absolute position: 0s

// Labels
tl.addLabel('phase2', 2)
tl.from('.cards', { y: 30 }, 'phase2')
tl.from('.cards', { y: 30 }, 'phase2+=0.5')  // 0.5s after label
```

### Timeline Position Parameter

| Value | Meaning |
|---|---|
| `0` | Absolute time: 0 seconds |
| `1.5` | Absolute time: 1.5s |
| `'-=0.5'` | 0.5s before end of timeline |
| `'+=0.2'` | 0.2s after end of timeline |
| `'<'` | Same start as previous tween |
| `'<0.3'` | 0.3s after previous tween starts |
| `'>-0.2'` | 0.2s before previous tween ends |
| `'labelName'` | At label |
| `'labelName+=1'` | 1s after label |

---

## Eases

### Built-in families
```
power1, power2, power3, power4
back, elastic, bounce, circ, expo, sine
steps(n)
```

Each has `.in`, `.out`, `.inOut`:
```js
ease: 'power3.out'     // most common for entrances
ease: 'power3.in'      // exits
ease: 'power3.inOut'   // symmetric
ease: 'back.out(1.7)'  // overshoot amount
ease: 'elastic.out(1, 0.3)'  // amplitude, period
ease: 'steps(5)'       // stepped, like CSS steps()
ease: 'none'           // linear
```

### ExpoScale (great for scale transitions)
```js
ease: 'expoScale(1, 5)'          // scale 1→5, linear time
ease: 'expoScale(0.1, 1, power2)' // with nested ease
```

### CustomEase
```js
import { CustomEase } from 'gsap/CustomEase'
CustomEase.create('myEase', 'M0,0 C0.14,0 0.242,0.438 0.272,0.561 ...')
gsap.to(el, { x: 100, ease: 'myEase' })
```

### CustomBounce / CustomWiggle / RoughEase / SlowMo
```js
// CustomBounce
CustomBounce.create('myBounce', { strength: 0.6, endAtStart: false })
gsap.to(el, { y: 100, ease: 'myBounce' })

// RoughEase (Club)
ease: RoughEase.ease.config({ strength: 3, points: 20, randomize: true })

// SlowMo — slow in middle, fast at ends
ease: 'slow(0.7, 0.7, false)'
```

---

## Staggers

```js
// Simple
gsap.from('.item', { y: 40, opacity: 0, stagger: 0.1 })

// Advanced stagger object
gsap.from('.item', {
  y: 40,
  opacity: 0,
  stagger: {
    each: 0.1,          // time between each
    from: 'center',     // start from center: 'start', 'end', 'center', 'edges', 'random', index
    grid: 'auto',       // for grid layouts: 'auto' or [rows, cols]
    axis: 'x',          // for grid: 'x', 'y', or null
    ease: 'power2.inOut',
    repeat: -1,
    yoyo: true,
  }
})

// Function-based (index, target, list)
gsap.from('.item', {
  y: (i) => i * 20,
  opacity: 0,
  delay: (i) => i * 0.05,
})
```

---

## Utility Methods

```js
// Wrap — loop through array indices
const wrap = gsap.utils.wrap(0, items.length)
wrap(items.length + 1) // → 1

// Clamp — constrain value
const clamp = gsap.utils.clamp(-20, 20)
clamp(50) // → 20

// Interpolate — lerp between values
const lerp = gsap.utils.interpolate(0, 100, 0.5) // → 50
gsap.utils.interpolate(['red', 'blue'], 0.5) // color interpolation
gsap.utils.interpolate([0, 100, 200], 0.75) // between array items

// MapRange — remap a value from one range to another
const map = gsap.utils.mapRange(0, 1, 0, 100)
map(0.5) // → 50

// Normalize — map 0-1 from a range
const norm = gsap.utils.normalize(0, 100)
norm(50) // → 0.5

// Snap — snap to nearest increment
const snap = gsap.utils.snap(0.25)
snap(0.3) // → 0.25
gsap.utils.snap([0, 0.1, 0.5, 1], 0.35) // snap to nearest in array

// Random
gsap.utils.random(0, 100)
gsap.utils.random(0, 100, true) // return reusable function
gsap.utils.random(['red', 'green', 'blue'])

// ToArray — normalize selector to array
gsap.utils.toArray('.item') // → HTMLElement[]
gsap.utils.toArray(nodeList)

// Selector — scoped querySelector
const q = gsap.utils.selector(containerRef)
q('.item') // only selects .item within containerRef

// Shuffle
gsap.utils.shuffle([1, 2, 3, 4, 5])

// Pipe — compose utility functions
const transform = gsap.utils.pipe(
  gsap.utils.clamp(0, 100),
  gsap.utils.mapRange(0, 100, 0, 1),
)
```

---

## gsap.quickTo() — Smooth Cursor Tracking

Creates a function that re-targets an active tween. Best for smooth cursor/mouse animations.

```js
// Create reusable tweeners
const setX = gsap.quickTo(el, 'x', { duration: 0.4, ease: 'power3' })
const setY = gsap.quickTo(el, 'y', { duration: 0.4, ease: 'power3' })

// On mousemove — no tween creation overhead
document.addEventListener('mousemove', (e) => {
  setX(e.clientX)
  setY(e.clientY)
})

// With optional start value (for instant jump on first enter)
setX(e.clientX, e.clientX) // second arg = start value
```

## gsap.quickSetter() — Fastest Property Updates

For per-frame updates (ticker, scroll handler) where you need raw speed:

```js
const skewSetter = gsap.quickSetter('.skewElem', 'skewY', 'deg')
// Then call synchronously with no tweening:
skewSetter(10) // instantly sets skewY: 10deg
```

---

## gsap.ticker — Frame Loop

```js
// Add function to GSAP's RAF loop
gsap.ticker.add((time, deltaTime, frame) => {
  // Called every frame
  // time = elapsed seconds
  // deltaTime = ms since last frame
})

// Remove
gsap.ticker.remove(myFunc)

// FPS cap
gsap.ticker.fps(60)

// Lag smoothing (prevents jumps after tab switch)
gsap.ticker.lagSmoothing(500, 33)
```

---

## gsap.matchMedia() — Responsive

```js
const mm = gsap.matchMedia()

mm.add({
  isDesktop: '(min-width: 1024px)',
  isTablet: '(min-width: 768px) and (max-width: 1023px)',
  isMobile: '(max-width: 767px)',
  reduceMotion: '(prefers-reduced-motion: reduce)',
}, (context) => {
  const { isDesktop, isMobile, reduceMotion } = context.conditions

  if (reduceMotion) {
    // Skip or simplify animations
    return
  }

  if (isDesktop) {
    gsap.from('.hero', { y: 100, opacity: 0, duration: 1.2 })
  } else {
    gsap.from('.hero', { opacity: 0, duration: 0.6 })
  }
})
```

---

## gsap.context() — Scoping

```js
const ctx = gsap.context(() => {
  gsap.from('.item', { y: 40, opacity: 0 })
  ScrollTrigger.create({ ... })
  // All tweens and ScrollTriggers registered here are tracked
}, containerElement) // optional scope element

// Clean everything up
ctx.revert()

// Add more animations later
ctx.add(() => {
  gsap.to('.other', { x: 100 })
})
```

---

## Tween Control

```js
const tween = gsap.to('.box', { x: 200, duration: 2, paused: true })

tween.play()
tween.pause()
tween.reverse()
tween.restart()
tween.seek(1)          // jump to 1 second
tween.progress(0.5)    // jump to 50%
tween.timeScale(2)     // 2x speed
tween.kill()
tween.invalidate()     // re-read starting values (after DOM change)
tween.isActive()       // boolean
tween.then(() => {})   // Promise-like
```

---

## Kill & Cleanup

```js
gsap.killTweensOf('.box')         // kill all tweens on .box
gsap.killTweensOf('.box', 'x,y')  // kill only x and y tweens
gsap.set('.box', { clearProps: 'all' })  // remove all inline styles GSAP set
gsap.set('.box', { clearProps: 'x,y,rotation' })
```

---

## gsap.defaults()

```js
// Set defaults for ALL tweens in this project
gsap.defaults({
  ease: 'power3.out',
  duration: 0.8,
  overwrite: 'auto',  // automatically kill conflicting tweens
})
```

---

## gsap.config()

```js
gsap.config({
  force3D: true,         // always use 3D transforms
  nullTargetWarn: false, // suppress null target warnings
  trialWarn: false,      // suppress trial plugin warnings
  units: { x: 'vw' },   // default units per property
})
```
