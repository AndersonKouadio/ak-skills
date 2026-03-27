# Canvas & WebGL Patterns

## Particle Canvas (Example 8)

GSAP timeline drives canvas 2D particles (no DOM elements):

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import gsap from 'gsap'
import { useRef, useEffect } from 'react'

const PARTICLE_COUNT = 99
const IMAGE_SRCS = Array.from({ length: 21 }, (_, i) => `/particles/flair-${i + 2}.png`)

export function ParticleCanvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null)

  useGSAP(() => {
    const canvas = canvasRef.current!
    const ctx = canvas.getContext('2d')!
    let cw = (canvas.width = window.innerWidth)
    let ch = (canvas.height = window.innerHeight)
    let radius = Math.max(cw, ch)

    type Particle = { x: number; y: number; scale: number; rotate: number; img: HTMLImageElement }
    const particles: Particle[] = Array.from({ length: PARTICLE_COUNT }, (_, i) => ({
      x: 0, y: 0, scale: 0, rotate: 0,
      img: Object.assign(new Image(), { src: IMAGE_SRCS[i % IMAGE_SRCS.length] }),
    }))

    function draw() {
      particles.sort((a, b) => a.scale - b.scale)
      ctx.clearRect(0, 0, cw, ch)
      particles.forEach((p) => {
        ctx.translate(cw / 2, ch / 2)
        ctx.rotate(p.rotate)
        ctx.drawImage(p.img, p.x, p.y, p.img.width * p.scale, p.img.height * p.scale)
        ctx.resetTransform()
      })
    }

    const tl = gsap.timeline({ onUpdate: draw })
      .fromTo(particles,
        {
          x: (i) => Math.cos((i / PARTICLE_COUNT * Math.PI * 2) - Math.PI / 2 * 10) * radius,
          y: (i) => Math.sin((i / PARTICLE_COUNT * Math.PI * 2) - Math.PI / 2 * 10) * radius,
          scale: 1.1, rotate: 0,
        },
        {
          duration: 5, ease: 'sine',
          x: 0, y: 0, scale: 0, rotate: -3,
          stagger: { each: -0.05, repeat: -1 },
        },
        0
      )
      .seek(99)

    const handleResize = () => {
      cw = canvas.width = window.innerWidth
      ch = canvas.height = window.innerHeight
      radius = Math.max(cw, ch)
      tl.invalidate()
    }
    window.addEventListener('resize', handleResize)

    // Toggle play/pause on click
    canvas.addEventListener('pointerup', () => {
      gsap.to(tl, { timeScale: tl.isActive() ? 0 : 1 })
    })

    return () => window.removeEventListener('resize', handleResize)
  })

  return (
    <canvas
      ref={canvasRef}
      style={{ display: 'block', width: '100vw', height: '100vh' }}
    />
  )
}
```

---

## Scroll Velocity → WebGL Shader (Example 23)

GSAP feeds scroll velocity into Three.js shader uniforms:

```tsx
'use client'
import { useGSAP } from '@gsap/react'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import gsap from 'gsap'
import * as THREE from 'three'

gsap.registerPlugin(ScrollTrigger)

const VERT = /* glsl */`
  varying vec2 vUv;
  varying vec2 vUvCover;
  uniform vec2 uTextureSize;
  uniform vec2 uQuadSize;

  void main(){
    vUv = uv;
    float texR = uTextureSize.x / uTextureSize.y;
    float quadR = uQuadSize.x / uQuadSize.y;
    vec2 s = vec2(1.0);
    if (quadR > texR) { s.y = texR / quadR; } else { s.x = quadR / texR; }
    vUvCover = vUv * s + (1.0 - s) * 0.5;
    gl_Position = vec4(position, 1);
  }
`

const FRAG = /* glsl */`
  precision highp float;
  uniform sampler2D uTexture;
  uniform float uTime;
  uniform float uScrollVelocity;
  uniform float uVelocityStrength;
  varying vec2 vUvCover;

  void main() {
    vec2 tc = vUvCover;
    float amt = 0.03 * uVelocityStrength;
    float t = uTime * 0.8;
    tc.y += sin((tc.x * 8.0) + t) * amt;
    tc.x += cos((tc.y * 6.0) - t * 0.8) * amt * 0.6;
    float dir = sign(uScrollVelocity);
    float r = texture2D(uTexture, tc + vec2( amt * 0.50 * dir, 0.0)).r;
    float g = texture2D(uTexture, tc + vec2( amt * 0.25 * dir, 0.0)).g;
    float b = texture2D(uTexture, tc + vec2(-amt * 0.35 * dir, 0.0)).b;
    gl_FragColor = vec4(r, g, b, 1.0);
  }
`

// Shared velocity proxy — one ScrollTrigger drives all canvases
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

export function ShaderFrame({ imageUrl }: { imageUrl: string }) {
  const frameRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const frameEl = frameRef.current!
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
    frameEl.appendChild(renderer.domElement)
    renderer.outputColorSpace = THREE.SRGBColorSpace
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

    const scene = new THREE.Scene()
    const camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1)
    const geom = new THREE.PlaneGeometry(2, 2)
    const uniforms = {
      uTexture: { value: null as THREE.Texture | null },
      uTextureSize: { value: new THREE.Vector2(1, 1) },
      uQuadSize: { value: new THREE.Vector2(1, 1) },
      uTime: { value: 0 },
      uScrollVelocity: { value: 0 },
      uVelocityStrength: { value: 0 },
    }

    const mat = new THREE.ShaderMaterial({ uniforms, vertexShader: VERT, fragmentShader: FRAG, transparent: true })
    scene.add(new THREE.Mesh(geom, mat))

    const loader = new THREE.TextureLoader()
    loader.load(imageUrl, (tex) => {
      tex.colorSpace = THREE.SRGBColorSpace
      uniforms.uTexture.value = tex
      uniforms.uTextureSize.value.set(tex.image.width, tex.image.height)
      layout()
    })

    function layout() {
      const { width, height } = frameEl.getBoundingClientRect()
      renderer.setSize(width, height, false)
      uniforms.uQuadSize.value.set(width, height)
    }

    let last = performance.now()
    function tick(now: number) {
      const dt = (now - last) * 0.001; last = now
      uniforms.uTime.value += dt
      uniforms.uScrollVelocity.value = velocityProxy.v
      uniforms.uVelocityStrength.value = velocityProxy.s
      renderer.render(scene, camera)
    }
    gsap.ticker.add(tick)

    return () => {
      gsap.ticker.remove(tick)
      renderer.dispose()
    }
  }, [imageUrl])

  return (
    <div
      ref={frameRef}
      style={{ position: 'relative', width: '80vw', maxWidth: 600, aspectRatio: '16/9', margin: '20vh auto' }}
    />
  )
}
```

---

## Three.js + Flip Waypoints (Example 24)

GSAP `Flip.fit()` moves a Three.js canvas between DOM markers while the 3D object rotates:

Key pattern:
1. Create `<canvas>` element in starting container
2. Capture marker states with `Flip.getState()`
3. Use `Flip.fit(canvas, markerState)` in a timeline
4. Run Three.js rotation in parallel with `< `

See [flip.md](../flip.md) for full implementation.

---

## GSAP + Canvas Performance Tips

1. **Use `gsap.ticker.add()`** — runs in sync with GSAP's RAF, better than `requestAnimationFrame` directly
2. **`tl.seek(n)`** — jump to frame n on init (prevents flash before animation starts)
3. **`tl.invalidate()`** — call on resize to recalculate function-based start values
4. **Sort particles by scale** before drawing to fake z-depth: `particles.sort((a,b) => a.scale - b.scale)`
5. **`ctx.resetTransform()`** instead of `ctx.restore()` — faster
6. **Toggle play with timeScale**: `gsap.to(tl, { timeScale: tl.isActive() ? 0 : 1 })`
