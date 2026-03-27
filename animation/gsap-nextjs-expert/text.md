# Text Animations

## SplitText (Club Plugin)

SplitText breaks text into animatable units. **Always wait for fonts to load.**

```js
gsap.registerPlugin(SplitText)

// Basic split
const split = SplitText.create('.headline', {
  type: 'chars,words,lines',  // what to split into
  charsClass: 'char',
  wordsClass: 'word',
  linesClass: 'line',
  mask: 'lines',              // adds overflow:hidden mask wrappers around each line
  autoSplit: true,            // re-split on resize automatically
  onSplit: (instance) => {
    // Called on initial split AND on every resize
    // MUST return an animation for autoSplit to work correctly
    return gsap.from(instance.lines, {
      yPercent: 120,
      stagger: 0.08,
      duration: 1,
      ease: 'power3.out',
    })
  }
})

// Access split elements
split.chars   // array of char elements
split.words   // array of word elements
split.lines   // array of line elements

// Revert (restore original HTML)
split.revert()

// Manual split/re-split
split.split({ type: 'chars' })
```

---

## Responsive Line Splits with ScrollTrigger (Example 30)

The correct pattern — `autoSplit: true` + return animation from `onSplit`:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { SplitText } from 'gsap/SplitText'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'
import { useRef } from 'react'

gsap.registerPlugin(SplitText, ScrollTrigger)

export function TextReveal({ children }: { children: React.ReactNode }) {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    document.fonts.ready.then(() => {
      const texts = gsap.utils.toArray<HTMLElement>('.split-text', container.current)

      texts.forEach((text) => {
        SplitText.create(text, {
          type: 'words,lines',
          mask: 'lines',
          linesClass: 'line',
          autoSplit: true,
          onSplit: (instance) => {
            return gsap.from(instance.lines, {
              yPercent: 120,
              stagger: 0.08,
              scrollTrigger: {
                trigger: text,
                scrub: true,
                start: 'clamp(top center)',
                end: 'clamp(bottom center)',
              }
            })
          }
        })
      })
    })
  }, { scope: container })

  return <div ref={container}>{children}</div>
}
```

---

## Character Reveal — Stagger Entrance

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const split = SplitText.create('.hero-title', {
      type: 'chars,words',
      charsClass: 'char',
    })

    gsap.from(split.chars, {
      y: 80,
      opacity: 0,
      rotationX: -90,
      stagger: {
        each: 0.03,
        from: 'random',
        ease: 'power2.inOut',
      },
      duration: 0.8,
      ease: 'back.out(1.7)',
      transformOrigin: '0% 50% -50px',
    })
  })
}, { scope: container })
```

---

## Rolling Tube Text (Example 9)

3D cylinder rotation effect — each set of chars rolls through rotationX:

```tsx
useGSAP(() => {
  const lines = document.querySelectorAll('.line')
  const depth = -(window.innerWidth / 8)
  const transformOrigin = `50% 50% ${depth}`

  gsap.set(lines, { perspective: 700, transformStyle: 'preserve-3d' })

  const tl = gsap.timeline({ repeat: -1 })

  const splitLines = Array.from(lines).map(line =>
    SplitText.create(line, { type: 'chars' })
  )

  splitLines.forEach((split, index) => {
    tl.fromTo(
      split.chars,
      { rotationX: -90 },
      {
        rotationX: 90,
        stagger: 0.08,
        duration: 0.9,
        ease: 'none',
        transformOrigin,
      },
      index * 0.45
    )
  })
}, { scope: container })
```

CSS required:
```css
.tube {
  height: 24vw;    /* visible window — clip overflow */
  overflow: hidden;
}
.line {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  white-space: nowrap;
}
.line div { backface-visibility: hidden; }
```

---

## Horizontal Scroll Text (Example 13)

Text scrolls horizontally while chars animate in from random positions:

```tsx
useGSAP(() => {
  const wrapper = document.querySelector('.Horizontal')
  const text = document.querySelector('.Horizontal__text')
  const split = SplitText.create('.Horizontal__text', { type: 'chars, words' })

  // Parent horizontal scroll
  const scrollTween = gsap.to(text, {
    xPercent: -100,
    ease: 'none',
    scrollTrigger: {
      trigger: wrapper,
      pin: true,
      end: '+=5000px',
      scrub: true,
    }
  })

  // Each char enters from random vertical offset
  split.chars.forEach((char) => {
    gsap.from(char, {
      yPercent: 'random(-200, 200)',
      rotation: 'random(-20, 20)',
      ease: 'back.out(1.2)',
      scrollTrigger: {
        trigger: char,
        containerAnimation: scrollTween,
        start: 'left 100%',
        end: 'left 30%',
        scrub: 1,
      }
    })
  })
}, { scope: container })
```

CSS:
```css
.Horizontal {
  overflow: hidden;
  height: 100vh;
  display: flex;
  align-items: center;
}
.Horizontal__text {
  display: flex;
  width: max-content;
  white-space: nowrap;
  gap: 4vw;
  padding-left: 100vw;
  font-size: clamp(2rem, 10vw, 12rem);
}
```

---

## Word-by-Word Reveal

```tsx
useGSAP(() => {
  document.fonts.ready.then(() => {
    const split = SplitText.create('.body-text', {
      type: 'words',
      mask: 'words',
    })

    gsap.from(split.words, {
      opacity: 0,
      y: 20,
      stagger: 0.04,
      duration: 0.6,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: '.body-text',
        start: 'top 80%',
        once: true,
      }
    })
  })
}, { scope: container })
```

---

## ScrambleText Plugin

```js
gsap.to('.scramble-text', {
  duration: 2,
  scrambleText: {
    text: 'Final revealed text',
    chars: 'upperCase',   // 'lowerCase', 'upperCase', custom string
    revealDelay: 0.5,
    speed: 0.5,
    delimiter: '',        // '' = char by char, ' ' = word by word
  }
})
```

---

## TextPlugin

```js
import { TextPlugin } from 'gsap/TextPlugin'
gsap.registerPlugin(TextPlugin)

gsap.to('.typewriter', {
  duration: 2,
  text: {
    value: 'Hello, World!',
    delimiter: '',   // type char by char
    padSpace: true,  // pad with spaces to maintain width
  },
  ease: 'none',
})
```

---

## Masking Lines Technique

For clean line reveals without overflow visible:

```css
/* SplitText mask: 'lines' adds these automatically */
.line-mask {
  overflow: hidden;
  display: block;
}

/* For manual control */
.text-container .word {
  overflow: hidden;
  display: inline-block;
  vertical-align: top;
}
```

```js
// Manual mask approach
const split = SplitText.create('.title', { type: 'lines', mask: 'lines' })
gsap.from(split.lines, { yPercent: 100, stagger: 0.1 })
// Lines slide up from underneath the mask
```
