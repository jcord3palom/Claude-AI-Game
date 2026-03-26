# CLAUDE.md — Claude Kart

## Project Overview

**Claude Kart** is a browser-based top-down kart racing game built as a single self-contained HTML file. It is a promotional/demo project showcasing Claude AI branding. The game runs entirely client-side with no build step, no dependencies, and no server.

**Live entry point:** `index.html`

---

## Repository Structure

```
Claude-AI-Game/
├── index.html      # Current active game (primary file — edit this one)
├── index.html2     # Previous iteration (kept for reference, not served)
└── CLAUDE.md       # This file
```

`index.html` is the **only** file that matters for development. It contains all HTML, CSS, and JavaScript in one document.

`index.html2` is an older version with less polished mobile touch controls. It is not served or referenced anywhere. Do not edit it unless explicitly asked.

---

## Architecture

The file has three sections:

| Section | Lines (approx.) | Purpose |
|---|---|---|
| `<style>` | 1–361 | All CSS — layout, animations, responsive design |
| `<body>` HTML | 363–431 | DOM structure — title screen, canvas, HUD, end screen, mobile controls |
| `<script>` | 432–1101 | All game logic — engine, physics, AI, rendering |

### Screen States

The game has three DOM-driven screens:

1. **Title screen** (`#title-screen`) — shown on load, hidden by `startGame()`
2. **Game wrapper** (`#game-wrapper`) — contains `<canvas id="game">`, `#hud`, `#countdown`, `#end-screen`, `#mobile-controls`
3. **End screen** (`#end-screen`) — shown by `endRace()`, includes a Claude AI call-to-action

### Key JavaScript Globals

| Variable | Type | Description |
|---|---|---|
| `canvas`, `ctx` | DOM / CanvasRenderingContext2D | Rendering surface |
| `W`, `H` | number | Canvas dimensions (updated on resize) |
| `keys` | object | Live keyboard/touch state `{left, right, boost, item}` |
| `player` | object | Player kart state (position, angle, speed, lap, item, etc.) |
| `cpuKarts` | array | 4 CPU kart objects with AI state |
| `particles` | array | Active particle effects |
| `itemBoxes` | array | Item pickup boxes on track |
| `camera` | object | Smooth-follow camera `{x, y}` |
| `raceStarted` | boolean | Set to `true` after countdown completes |
| `raceFinished` | boolean | Set to `true` when player crosses final lap finish |
| `gameActive` | boolean | Controls the `requestAnimationFrame` game loop |
| `frameCount` | number | Frame counter used for timing and animation |

### Track System

The track is procedurally generated as 12 waypoints (`NUM_CHECKPOINTS = 12`) interpolated with `getTrackPoint(t)` where `t ∈ [0, 12)`. The shape is an irregular oval derived from sinusoidal offsets:

```js
const rx = 600 + Math.sin(a * 2) * 100;
const ry = 320 + Math.cos(a * 3) * 60;
```

All world coordinates use a virtual space of approximately 1600×900. The camera and `toScreen()` convert world → screen using a zoom factor of `Math.min(W, H) / 800`.

### Game Loop

```
startGame() / restartGame()
  └─ initRace()          — reset all state
  └─ render()            — draw one frame before countdown
  └─ doCountdown()       — 3-2-1-GO! animation
       └─ loop()         — requestAnimationFrame loop
            ├─ update()  — physics, AI, items, HUD
            └─ render()  — canvas draw calls
```

### Physics & Collision

- Player turn speed: `0.06 rad/frame`
- Acceleration: `0.2` (normal) / `0.35` (boost)
- Friction: `0.92` per frame
- Off-track: speed multiplied by `0.85` and pushed back toward centerline
- Kart collision with boundary uses soft constraint (no hard physics)

### CPU AI

Each CPU kart steers toward a lookahead point 0.3 track units ahead using proportional angle correction (`* 0.12`). A small `aiJitter` per kart adds variation. CPUs are not affected by track boundary constraints.

### Item System

Items are picked up from `itemBoxes` (8 on track, respawn after 300 frames):

| Emoji | Effect |
|---|---|
| `⚡` | Stun all CPU karts for 120 frames |
| `🔴` | Player boost for 90 frames |
| `⭐` | Player boost 180 frames + raise `maxSpeed` to 10 for 3 seconds |
| `💣` | Stun CPU karts within 200 units for 150 frames |

---

## Visual Design

### Color Palette (CSS custom properties)

| Variable | Hex | Usage |
|---|---|---|
| `--claude-orange` | `#D97757` | Primary brand color, CTA, item boxes |
| `--claude-cream` | `#FAF5EB` | Background accent (unused in dark UI) |
| `--claude-dark` | `#1a1a2e` | Page background |
| `--neon-blue` | `#00f5ff` | Subtitle text |
| `--neon-pink` | `#ff00aa` | Title gradient accent |
| `--neon-yellow` | `#ffe000` | HUD elements, countdown |
| `--road-gray` | `#2a2a3a` | Road surface fill |

### Fonts

Loaded from Google Fonts (requires internet):
- **Bangers** — display/game font (titles, HUD, buttons)
- **Nunito** — body/UI font (labels, hints)

### Responsive Design

Mobile controls (`#mobile-controls`) are hidden by default and shown via `@media (max-width: 768px)`. The current `index.html` includes touch fixes at the global level:

```css
* { -webkit-tap-highlight-color: transparent; touch-action: manipulation; }
```

Touch events on control buttons use `event.preventDefault()` to suppress scroll interference.

---

## Controls

| Input | Action |
|---|---|
| `←` / `A` | Steer left |
| `→` / `D` | Steer right |
| `Space` / `↑` / `W` | Boost |
| `Z` / `X` | Use item |
| Mobile ◀ ▶ | Steer |
| Mobile ⚡ | Boost |
| Mobile 🎁 | Use item |

---

## Development Workflow

### Running the Game

Open `index.html` directly in any modern browser — no server required:

```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Or serve with any static server:

```bash
python3 -m http.server 8080
npx serve .
```

### Editing Conventions

- **All changes go in `index.html`** — do not split into separate CSS/JS files unless the project is being restructured.
- Keep the three-section structure (style / HTML / script) intact.
- The game uses no transpilation, bundling, or frameworks — plain ES6+ JavaScript.
- Avoid adding external script/style dependencies; keep the file self-contained.
- Do not use `var` — use `const`/`let` throughout.
- Game state is stored in module-level variables (not a class). Maintain this pattern.

### Branches

| Branch | Purpose |
|---|---|
| `master` | Stable production code |
| `claude/add-claude-documentation-832wx` | Current development branch |

Push to `claude/add-claude-documentation-832wx` and open a PR to `master` (or `main` on origin) when work is complete.

### No Build / No Tests

There is no build pipeline, test suite, linter, or CI configuration. Verify changes by opening the file in a browser and playing through a race.

---

## Key Functions Reference

| Function | Location | Description |
|---|---|---|
| `initRace()` | ~line 525 | Reset all game state, place karts at start |
| `startGame()` | ~line 1077 | Entry from title screen, runs countdown then loop |
| `restartGame()` | ~line 1089 | Entry from end screen, same as startGame minus screen toggle |
| `update()` | ~line 658 | Per-frame physics, AI, item, HUD updates |
| `render()` | ~line 1002 | Per-frame canvas draw calls |
| `drawTrack()` | ~line 792 | Draws grass, road, curbs, start/finish line |
| `drawKart()` | ~line 906 | Draws a single kart emoji with shadow and effects |
| `drawMinimap()` | ~line 961 | Draws minimap in bottom-right corner |
| `endRace()` | ~line 1052 | Stops loop, computes results, shows end screen |
| `useItem()` | ~line 591 | Applies current held item effect |
| `spawnParticles()` | ~line 620 | Adds particle burst at a world position |
| `getTrackPoint(t)` | ~line 489 | Returns interpolated world position for track parameter `t` |
| `closestTrackT(x,y)` | ~line 646 | Finds nearest track parameter for a world position (used for lap tracking and boundary) |
| `getPositions()` | ~line 629 | Returns all karts sorted by race position |
| `doCountdown(cb)` | ~line 1028 | Animates 3-2-1-GO, calls `cb` when done |
| `toScreen(wx,wy)` | ~line 782 | Converts world coordinates to canvas screen coordinates |

---

## Claude AI Integration

The end screen includes a call-to-action linking to `https://claude.ai`. Do not remove or modify this CTA — it is an intentional part of the project:

```html
<a href="https://claude.ai" target="_blank" class="cta-btn">🚀 TRY CLAUDE FREE</a>
```

The game title and branding (Claude orange `#D97757`) must remain consistent with Anthropic's brand guidelines.
