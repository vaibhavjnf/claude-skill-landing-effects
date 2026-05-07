# Skill: landing-effects

You have access to the `landing-effects` npm package — a zero-dependency, framework-agnostic TypeScript library that provides two canvas-based visual effects for landing pages.

## Install

```bash
npm install landing-effects
# or
bun add landing-effects
```

## Effects Overview

| Effect | Function | What it does |
|--------|----------|--------------|
| ASCII Renderer | `createAsciiRenderer` | Converts an image to interactive ASCII art with mouse parallax, glitch bands, edge-inward reveal, and depth-based coloring |
| Pixel Reveal | `createPixelReveal` | Randomized block-by-block image reveal → glitch refine → crisp final |

Both functions return a `cleanup()` function. Always call it on component unmount / page teardown to stop animation loops and remove event listeners.

---

## Effect 1 — ASCII Renderer

### Basic usage

```ts
import { createAsciiRenderer } from 'landing-effects'

const canvas = document.getElementById('ascii-canvas') as HTMLCanvasElement

const cleanup = createAsciiRenderer({
  canvas,
  imageSrc: '/hero.png',
})

// Teardown
// cleanup()
```

### All options

```ts
createAsciiRenderer({
  // REQUIRED
  canvas: HTMLCanvasElement,     // target canvas element
  imageSrc: string,              // image URL (local or remote)

  // OPTIONAL — with defaults
  chars: ' 0123456789',          // character ramp, dark → light
  fontSize: 9,                   // font size in px
  fontFamily: '"DM Mono", monospace',
  brightnessBoost: 2.2,          // brightness multiplier
  posterize: 32,                 // posterization steps (lower = more blocky)
  parallaxStrength: 8,           // mouse parallax intensity (0 = disabled)
  scale: 1.15,                   // image zoom factor
  colorFn: (luminance: number, distFromCenter: number) => string, // custom CSS color per char
})
```

### colorFn example (custom blue tint)

```ts
colorFn: (lum, dist) => {
  const r = Math.floor(lum * 0.4 * 255)
  const g = Math.floor(lum * 0.6 * 255)
  const b = Math.floor((0.4 + lum * 0.6) * 255)
  return `rgb(${r},${g},${b})`
}
```

### Canvas sizing tip

The renderer automatically fills the canvas to its `offsetWidth` / `offsetHeight`. Set size via CSS:

```html
<canvas id="ascii-canvas" style="width:100%;height:100vh;"></canvas>
```

---

## Effect 2 — Pixel Reveal

### Basic usage

```ts
import { createPixelReveal } from 'landing-effects'

const canvas = document.getElementById('reveal-canvas') as HTMLCanvasElement

const cleanup = createPixelReveal({
  canvas,
  imageSrc: '/product.png',
})
```

### All options

```ts
createPixelReveal({
  // REQUIRED
  canvas: HTMLCanvasElement,
  imageSrc: string,

  // OPTIONAL — with defaults
  blockSize: 8,            // pixel block size in px
  pixelsPerFrame: 120,     // blocks revealed per animation frame (higher = faster)
  glitchRegion: 0.36,      // fraction of height from top that gets glitch treatment (0–1)
  delay: 200,              // ms before animation starts
  onComplete: () => void,  // callback fired when reveal finishes
})
```

### glitchRegion examples

```ts
glitchRegion: 1/3   // top third gets glitch (good for faces/logos at top)
glitchRegion: 0.5   // top half
glitchRegion: 1     // entire image gets glitch treatment
glitchRegion: 0     // no glitch at all
```

### Canvas sizing tip

Set `width` and `height` attributes (not just CSS) so the pixel grid calculates correctly:

```html
<canvas id="reveal-canvas" width="400" height="500"></canvas>
```

---

## Framework Integration Examples

### React (with cleanup on unmount)

```tsx
import { useEffect, useRef } from 'react'
import { createAsciiRenderer, createPixelReveal } from 'landing-effects'

export function AsciiHero({ src }: { src: string }) {
  const canvasRef = useRef<HTMLCanvasElement>(null)

  useEffect(() => {
    if (!canvasRef.current) return
    const cleanup = createAsciiRenderer({
      canvas: canvasRef.current,
      imageSrc: src,
      parallaxStrength: 6,
    })
    return cleanup
  }, [src])

  return <canvas ref={canvasRef} style={{ width: '100%', height: '100vh' }} />
}

export function PixelRevealImage({ src }: { src: string }) {
  const canvasRef = useRef<HTMLCanvasElement>(null)

  useEffect(() => {
    if (!canvasRef.current) return
    const cleanup = createPixelReveal({
      canvas: canvasRef.current,
      imageSrc: src,
      glitchRegion: 1/3,
      onComplete: () => console.log('reveal done'),
    })
    return cleanup
  }, [src])

  return <canvas ref={canvasRef} width={600} height={400} />
}
```

### Vue 3 (Composition API)

```vue
<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import { createAsciiRenderer } from 'landing-effects'

const canvasRef = ref<HTMLCanvasElement | null>(null)
let cleanup: (() => void) | null = null

onMounted(() => {
  if (canvasRef.value) {
    cleanup = createAsciiRenderer({
      canvas: canvasRef.value,
      imageSrc: '/hero.png',
    })
  }
})

onUnmounted(() => cleanup?.())
</script>

<template>
  <canvas ref="canvasRef" style="width:100%;height:100vh" />
</template>
```

### Vanilla JS / Astro

```ts
import { createPixelReveal } from 'landing-effects'

const canvas = document.querySelector<HTMLCanvasElement>('#canvas')!
const cleanup = createPixelReveal({ canvas, imageSrc: '/hero.jpg' })

// For Astro: call cleanup in astro:before-swap
document.addEventListener('astro:before-swap', cleanup)
```

---

## Common Patterns

### Disable parallax (static ASCII)

```ts
createAsciiRenderer({ canvas, imageSrc, parallaxStrength: 0 })
```

### Fast pixel reveal

```ts
createPixelReveal({ canvas, imageSrc, pixelsPerFrame: 400, delay: 0 })
```

### Custom character set (denser ramp)

```ts
createAsciiRenderer({
  canvas,
  imageSrc,
  chars: ' .:-=+*#%@',
  fontSize: 7,
})
```

### Grayscale color function

```ts
createAsciiRenderer({
  canvas,
  imageSrc,
  colorFn: (lum) => `rgb(${Math.floor(lum*255)},${Math.floor(lum*255)},${Math.floor(lum*255)})`,
})
```

---

## Rules & Gotchas

- **Always call `cleanup()`** on component unmount or route change to prevent memory leaks and double animation loops.
- **CORS**: `imageSrc` must be on the same origin or served with `Access-Control-Allow-Origin: *`. Otherwise canvas `drawImage` will throw a security error.
- **Canvas must be in the DOM** before calling either function. Don't call inside SSR — guard with `typeof window !== 'undefined'`.
- For `createPixelReveal`, prefer **HTML attributes** (`width="600" height="400"`) over CSS for correct block grid math.
- For `createAsciiRenderer`, **CSS sizing** is fine because it reads `offsetWidth/offsetHeight`.
- Both effects use `requestAnimationFrame` internally — no setInterval/setTimeout loops to worry about.
- The library is **tree-shakeable** — importing only `createAsciiRenderer` won't bundle the pixel reveal code.

---

## Source

- GitHub: https://github.com/Dhravya/landing-effects
- npm: https://www.npmjs.com/package/landing-effects
- License: MIT
