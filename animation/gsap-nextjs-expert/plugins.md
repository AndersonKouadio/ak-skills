# Plugins Reference

## Plugin Registration Summary

```js
import gsap from 'gsap'
// Free plugins (included in gsap package)
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { Draggable } from 'gsap/Draggable'
import { MotionPathPlugin } from 'gsap/MotionPathPlugin'
import { Observer } from 'gsap/Observer'
import { TextPlugin } from 'gsap/TextPlugin'
import { ScrollToPlugin } from 'gsap/ScrollToPlugin'
import { EaselPlugin } from 'gsap/EaselPlugin'
import { PixiPlugin } from 'gsap/PixiPlugin'
import { Flip } from 'gsap/Flip'
import { CustomEase } from 'gsap/CustomEase'
import { CustomBounce } from 'gsap/CustomBounce'
import { CustomWiggle } from 'gsap/CustomWiggle'
import { ExpoScaleEase } from 'gsap/ExpoScaleEase'
import { RoughEase } from 'gsap/RoughEase'
import { SlowMo } from 'gsap/SlowMo'

// Club plugins (GSAP membership required)
import { SplitText } from 'gsap/SplitText'
import { ScrollSmoother } from 'gsap/ScrollSmoother'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import { DrawSVGPlugin } from 'gsap/DrawSVGPlugin'
import { GSDevTools } from 'gsap/GSDevTools'
import { InertiaPlugin } from 'gsap/InertiaPlugin'
import { Physics2DPlugin } from 'gsap/Physics2DPlugin'
import { PhysicsPropsPlugin } from 'gsap/PhysicsPropsPlugin'
import { ScrambleTextPlugin } from 'gsap/ScrambleTextPlugin'
import { MotionPathHelper } from 'gsap/MotionPathHelper'
import { CSSRulePlugin } from 'gsap/CSSRulePlugin'

gsap.registerPlugin(/* all of the above */)
```

---

## ScrollToPlugin

Animate the scroll position of any container:

```js
import { ScrollToPlugin } from 'gsap/ScrollToPlugin'
gsap.registerPlugin(ScrollToPlugin)

// Scroll window to element
gsap.to(window, { duration: 1, scrollTo: '#section2' })

// With offset
gsap.to(window, { duration: 1, scrollTo: { y: '#section2', offsetY: 100 } })

// Scroll to position
gsap.to(window, { duration: 1, scrollTo: 500 })

// Scroll a specific container
gsap.to('.scroll-container', { duration: 0.5, scrollTo: { y: 200, x: 0 } })

// Config max duration
gsap.to(window, {
  scrollTo: '#target',
  duration: gsap.utils.clamp(0.3, 1.5, distance / 1000),
})
```

---

## GSDevTools (Club)

Visual timeline debugger — DEV ONLY:

```js
import { GSDevTools } from 'gsap/GSDevTools'
gsap.registerPlugin(GSDevTools)

// Mount the UI (renders a draggable overlay)
GSDevTools.create({
  animation: myTimeline,  // optional — target a specific timeline
  minimal: false,
  css: true,              // inject default styles
  paused: false,
  keyboard: true,         // arrow keys to scrub
  id: 'dev-tools',
})

// Only include in dev
if (process.env.NODE_ENV === 'development') {
  GSDevTools.create()
}
```

---

## CSSRulePlugin

Animate CSS rules (pseudo-elements, etc.):

```js
import { CSSRulePlugin } from 'gsap/CSSRulePlugin'
gsap.registerPlugin(CSSRulePlugin)

// Get ::before or ::after rule
const rule = CSSRulePlugin.getRule('.button::before')

gsap.to(rule, {
  cssRule: { backgroundColor: 'red', width: '100%' },
  duration: 0.5,
})
```

---

## Physics2DPlugin

2D physics with gravity, velocity, acceleration, and friction:

```js
gsap.to('.particle', {
  duration: 2,
  physics2D: {
    velocity: 300,      // initial velocity (px/s)
    angle: -60,         // launch angle in degrees
    gravity: 500,       // gravity (px/s²)
    friction: 0.1,      // air friction (0-1)
  }
})

// Multi-directional burst
gsap.utils.toArray('.particle').forEach((el, i) => {
  gsap.to(el, {
    duration: 1.5 + Math.random(),
    physics2D: {
      velocity: 200 + Math.random() * 200,
      angle: i * (360 / particles.length),
      gravity: 400,
      friction: 0.05,
    },
    opacity: 0,
    delay: Math.random() * 0.3,
  })
})
```

---

## PhysicsPropsPlugin

Add physics to any numeric property:

```js
gsap.to('.element', {
  physicsProps: {
    x: { velocity: 200, friction: 0.1 },
    y: { velocity: -100, acceleration: 400, friction: 0 },
    rotation: { velocity: 360, friction: 0.05 },
  }
})
```

---

## ScrambleTextPlugin

Randomize characters during text reveal:

```js
import { ScrambleTextPlugin } from 'gsap/ScrambleTextPlugin'
gsap.registerPlugin(ScrambleTextPlugin)

gsap.to('.el', {
  duration: 2,
  scrambleText: {
    text: 'Target text here',
    chars: 'upperCase',    // 'lowerCase', 'upperCase', '01', 'XO#!', custom string
    revealDelay: 0.5,
    speed: 0.4,
    delimiter: '',         // '' = char, ' ' = word
    rightToLeft: false,
  }
})
```

---

## PixiPlugin

Animate Pixi.js display objects natively:

```js
import { PixiPlugin } from 'gsap/PixiPlugin'
import * as PIXI from 'pixi.js'
gsap.registerPlugin(PixiPlugin)
PixiPlugin.registerPIXI(PIXI) // required

const sprite = new PIXI.Sprite(texture)
app.stage.addChild(sprite)

gsap.to(sprite, {
  duration: 1,
  pixi: {
    x: 300,
    y: 200,
    alpha: 0.5,
    rotation: Math.PI,
    tint: 0xff0000,
    scale: 1.5,
    blur: 5,           // applies BlurFilter
    colorize: 'red',
    brightness: 2,
    saturation: 0.5,
    hue: 180,
    contrast: 1.5,
  }
})
```

---

## MotionPathHelper (Club)

Dev-time tool for visually editing motion paths:

```js
import { MotionPathHelper } from 'gsap/MotionPathHelper'
gsap.registerPlugin(MotionPathHelper)

// Edit an existing path in the browser
const helper = MotionPathHelper.editPath('#myPath', {
  selected: true,
  handleSize: 8,
})

// Kill helper
helper.kill()
```

---

## EaselPlugin

For CreateJS/EaselJS display objects:

```js
import { EaselPlugin } from 'gsap/EaselPlugin'
gsap.registerPlugin(EaselPlugin)

gsap.to(displayObject, {
  duration: 1,
  easel: {
    saturation: 0,
    brightness: 0.8,
    tint: '#ff0000',
    tintAmount: 0.5,
    colorize: '#00ff00',
    colorizeAmount: 1,
  }
})
```

---

## Ease Reference Card

| Ease | Best for |
|---|---|
| `power1.out` | Subtle, gentle |
| `power2.out` | Balanced, natural |
| `power3.out` | Most common entrance — premium feel |
| `power4.out` | Fast start, very slow landing |
| `expo.out` | Very dramatic deceleration |
| `back.out(1.7)` | Playful overshoot |
| `elastic.out(1, 0.3)` | Springy, bouncy |
| `bounce.out` | Ball bounce |
| `sine.inOut` | Most organic, wave-like |
| `none` / `linear` | Constant speed (scroll scrub, canvas) |
| `expoScale(1, 5)` | Scale transitions |
| `slow(0.7, 0.7)` | Slow in middle |
| `steps(5)` | Frame-by-frame |

### Choosing Ease by Brand Feel

| Brand | Ease Family |
|---|---|
| Premium / editorial | `power3.out`, `expo.out`, `sine.inOut` |
| Playful / friendly | `back.out`, `elastic.out` |
| Technical / minimal | `power2.out`, `none` |
| Energetic / sporty | `power4.out`, `expo.inOut` |
| Organic / natural | `sine.inOut`, `power2.inOut` |
