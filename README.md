# Girih-2-Generative-Art

A seed-based generative art catalogue for **Girih 2**, a computational Islamic-geometry system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Girih-2-Generative-Art/`

---

## Overview

**Girih 2** renders historic Islamic geometric patterns from numeric seeds. Each composition pairs a tessellating girih lattice — drawn from one of three historic pattern families — with a recursive tree rising behind it, in a curated palette of historic CSS colour names.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical pattern.

---

## Features

### Generative Engine (p5.js)

- **Three historic girih patterns**
  - **Umayyad mosque** — 12-point star tessellation
  - **Simple hexagram** — six-fold interlace with vertical offset rows
  - **Esreffoglu mosque** — six-petal rosette lattice with rotated lines
- **Recursive tree** — 12 levels deep, 1–4 sub-branches per node, drawn in a light companion colour against a saturated ground
- **Historic colour palettes**
  - 37 saturated ground colours (IndianRed, Crimson, Sienna, DarkSlateBlue…)
  - 28 pale companion colours (Lavender, MintCream, PowderBlue…)
  - 10 foreground pattern colours (RebeccaPurple, BlueViolet, DodgerBlue…)
- **Perfect-fit tiling** — pattern alignment calculated from canvas dimensions so the lattice covers edge to edge
- **Shadow blur per motif** — soft glow around each stroked motif, restored on each call so it doesn't leak

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop
- Shadow blur scoped per motif to avoid leaks across the tessellation

---

## File Structure

```
Girih-2-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Girih-2-Generative-Art.git
   cd Girih-2-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — ground colour, foreground colour, girih pattern, tree branching, motif scale — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Ground colour | 37 historic saturated CSS colours |
| Companion colour | 28 historic pale CSS colours |
| Foreground colour | 10 purples / blues |
| Girih pattern | Umayyad mosque / hexagram / Esreffoglu mosque |
| Motif scale `a` | 2–5% of canvas size |
| Tree depth | 12 levels |
| Tree sub-branches | 1–4 per node |
| Tree length decay | 0.6–1.0 of parent |

### The Girih Patterns

Each pattern is a tessellation of a single motif across a rectangular grid. The three patterns use different geometries and spacing rules:

**`star1` — Umayyad mosque**
- Motif: 12-point star, alternating outer radius `b` and inner radius `a`, where `b = a · sin(3π/4) / sin(π/12)`
- Grid: `dx = 2b`, `dy = 2b · cos(π/6)`
- Offset: odd rows shifted by `b` horizontally

**`star2` — Simple hexagram**
- Motif: 12-point star with outer radius `b = 2a · cos(π/6)`
- Grid: `dx = 3a`, `dy = 4a · cos(π/6)`
- Offset: even columns shifted by `dy / 2` vertically

**`star3` — Esreffoglu mosque**
- Motif: six-petal rosette of rotated line pairs at radius `a · cos(π/6) · 2`
- Grid: `dx = 6a · cos(π/6)`, `dy = 4.5a`
- Offset: odd rows shifted by `dx / 2` horizontally

All three compute an edge-aligned starting offset so the pattern begins and ends exactly at the canvas boundary — no gaps, no overflow.

### The Recursive Tree

Rising behind the pattern, drawn in the pale companion colour:

```
drawTree(x, y, length, angle, depth, width, color1, color2)
  ├── draw branch line from (0,0) to (0,-length)
  ├── if depth <= 1 → color1 (pale), else color2 (saturated)
  ├── branchWidth *= 0.9 per level
  └── for i in 1..4 sub-branches:
        newLength = length * (0.6 + rand * 0.4)
        newAngle = -2π + rand * 4π
        recurse at (0, -length) with newDepth = depth - 1
```

Twelve levels deep, with 1–4 children per node. Because each seed draws a different sequence of `rand()` calls, no two trees are alike — even within the same pattern family.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the colour palettes

Edit the arrays near the top of the script:

```js
const BG_COLOURS_1 = ['#CD5C5C', '#F08080', ...];   // 37 saturated grounds
const BG_COLOURS_2 = ['#FFFFE0', '#FFFACD', ...];   // 28 pale companions
const FG_COLOURS   = ['#663399', '#8A2BE2', ...];   // 10 foregrounds
```

Names are kept in parallel arrays (`BG_NAMES_1`, `BG_NAMES_2`, `FG_NAMES`) for the metadata display.

### Add a fourth girih pattern

1. Add a name to `GIRIH_NAMES`.
2. Write a `motif4` and `star4` following the shape of `motif3` / `star3`.
3. Add a branch in `drawGirih`:

```js
} else if (girih === 3) {
  const dx = /* your spacing */;
  const dy = /* your spacing */;
  const startX = -((cs / dx) % 1) * dx;
  const startY = -((cs / dy) % 1) * dy;
  p.translate(startX, startY);
  star4(p, a, cs + 2 * dy, cs + 2 * dx, color3, T, a * 2, sr);
}
```

4. Update the `sr.int(0, GIRIH_NAMES.length - 1)` call — it already uses `GIRIH_NAMES.length`, so no change needed there.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Portrait Series and Skyline Series templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live girih |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Girih-2-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **No `$fx` dependency** — the original calls `$fx.features()`. That's guarded with a `typeof $fx !== 'undefined'` check, so it silently skips in a plain browser.
2. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice.
3. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
4. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
5. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
6. **Drawing functions take `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
7. **Shadow blur scoped per motif** — set inside each motif function rather than globally, since `shadowBlur` otherwise leaks across the tessellation and dominates render time.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| **`Girih-2-Generative-Art`** | Islamic geometric ornament *(this repo)* |

All three share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Historic Patterns** — Umayyad mosque · Simple hexagram · Esreffoglu mosque
- **Edition** — Girih 2, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**