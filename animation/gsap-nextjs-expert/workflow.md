# Pre-Execution Workflow

Never write animation code without completing this analysis first.

---

## Step 1 — Classify the Request

**Is it a one-off component or a full-page animation project?**

| Signal | Type |
|---|---|
| "add a fade-in to this button" | Isolated component |
| "animate this page", "make the hero feel alive" | Page-level |
| "build the whole site animations" | Full project |
| "reproduce this video/reference" | Reproduction task |

---

## Step 2 — Explore the Project

If working inside a Next.js project, always read:

```
app/ or pages/         ← understand page structure
components/            ← which components exist
lib/ or utils/         ← existing animation utilities
package.json           ← which GSAP plugins are installed
tailwind.config        ← design tokens, colors, fonts
globals.css            ← CSS variables, brand palette
```

Look for:
- Existing GSAP setup or animation hooks
- Font loading strategy (important for SplitText)
- ScrollSmoother wrapper structure (`#smooth-wrapper` / `#smooth-content`)
- Brand colors, typography scale, spacing rhythm
- Whether the project uses App Router or Pages Router

---

## Step 3 — Ask the Right Questions

### For page-level or site-wide animations, always ask:

**1. Which sections/components need animation?**
> "I see a Hero, Features grid, Testimonials, and Footer. Should I animate all of them, or start with specific ones?"

**2. What's the brand feel?**
> "Is the brand more premium/editorial (slow, smooth, sophisticated eases) or energetic/playful (snappy, bouncy, elastic)?"

**3. Scroll interaction model?**
> "Should animations be scroll-triggered (scrub), triggered once on scroll (play on enter), or always playing?"

**4. Entry points?**
> "Should elements animate on page load (with a loader/intro sequence) or only as the user scrolls?"

**5. Mobile strategy?**
> "Should mobile have the same animations, simplified ones, or none? (Some heavy scroll effects degrade on mobile)"

**6. Existing plugins?**
> "Do you have a GSAP Club membership for access to SplitText, ScrollSmoother, Flip, MorphSVG? Or are you on the free tier?"

### For isolated component animations, minimal questions — just confirm:
- Target element selector
- Trigger condition (mount, hover, click, scroll)
- Desired visual effect if not clear

### For video/reference reproductions:
- Confirm the tech stack can support it (WebGL if needed, Canvas API)
- Identify which GSAP plugins map to the effect
- Note any assets needed (SVGs, images, fonts)

---

## Step 4 — Build the Animation Plan

Before writing code, outline:

```
## Animation Plan

### Plugins needed
- ScrollTrigger (scroll-driven)
- SplitText (text reveals)
- Flip (layout transitions)

### Component breakdown
- Hero: entrance sequence (chars split + fade up)
- Feature cards: stagger on scroll enter
- Testimonials: horizontal infinite scroll

### Performance notes
- Use `gsap.context()` for each component
- `will-change: transform` on animated elements
- `force3D: true` for heavy transform animations
- Lazy-register plugins only where used

### Brand alignment
- Ease: `power3.out` for entrances (matches premium feel)
- Duration: 0.8–1.2s for main elements
- Stagger: 0.08–0.15s between chars/items
```

---

## Step 5 — Choose the Right Tools

### Decision tree for plugin selection

```
Scroll-driven?
  ├─ Simple trigger (play on enter) → ScrollTrigger basic
  ├─ Scrub (tied to scroll position) → ScrollTrigger + scrub: true
  ├─ Smooth scroll + parallax → ScrollSmoother (Club)
  └─ Horizontal scroll section → ScrollTrigger + containerAnimation

Text animation?
  ├─ Split chars/words/lines → SplitText (Club)
  ├─ Scramble reveal → ScrambleTextPlugin
  └─ Simple type-on → TextPlugin

Layout transition (DOM reorder)?
  └─ Flip plugin → getState() → from() / to()

SVG?
  ├─ Path morphing → MorphSVGPlugin (Club)
  ├─ Draw on effect → DrawSVGPlugin (Club)
  └─ Move along path → MotionPathPlugin

Cursor / mouse interaction?
  ├─ Smooth tracking → quickTo()
  ├─ Trail effect → ticker + image pool
  ├─ Draggable element → Draggable + InertiaPlugin
  └─ Unified scroll/touch/pointer → Observer

Canvas?
  ├─ 2D particle/image effects → canvas 2D + gsap.ticker
  ├─ Image sequence on scroll → imageSequence helper
  └─ Three.js integration → gsap.ticker + Flip for position
```

---

## Step 6 — Confirm Before Coding

For complex tasks, present the plan to the user:

> "Here's my animation plan before I start coding:
> - **Hero**: SplitText character reveal on load with stagger
> - **Features**: Scroll-triggered stagger fade-up
> - **Footer**: MorphSVG bouncy entrance
>
> I'll need `SplitText`, `ScrollTrigger`, and `MorphSVGPlugin`. Do you have GSAP Club?
>
> Shall I proceed?"

For simple/obvious tasks, just start coding.

---

## Step 7 — Execute

Follow the implementation rules in:
- [nextjs-react.md](./nextjs-react.md) for React/Next.js safety
- [core-api.md](./core-api.md) for API correctness
- [patterns/](./patterns/) for proven implementations
