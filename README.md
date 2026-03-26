# PAC-MAN Simulation Game

A fully playable Pac-Man clone built in a single HTML file — no frameworks, no build tools, no dependencies.

Live demo: https://ycjanetchen.github.io/pacman_simugame/

---

## The Prompt

The original request was to build a browser-based Pac-Man game that:

- Renders a classic maze with walls, dots, and power pellets
- Lets the player control Pac-Man using arrow keys or WASD
- Includes ghost AI (Blinky, Pinky, Inky, Clyde) that chases the player
- Supports frightened mode when a power pellet is eaten
- Tracks score, high score, lives, and level
- Runs entirely in the browser with no server or build step

---

## Development Process

### 1. Maze Layout
The maze is defined as a 2D array (`BASE`) where each cell is a tile type:
- `0` = dot
- `1` = wall
- `2` = empty (no dot)
- `3` = power pellet
- `4` = ghost house door

The maze is drawn onto a `<canvas>` element each frame using the Canvas 2D API.

### 2. Movement System
Pac-Man and ghosts move on a pixel grid snapped to tile centers. A `near()` function detects when an entity is close enough to a tile center to allow a direction change or wall collision check.

**Key bug fixed during development:** The initial `near()` tolerance was `2.5px`, but Pac-Man's speed is `2.0px/frame`. This meant Pac-Man snapped back to center every frame and could never escape — appearing completely frozen. The fix was to pass `speed × 0.75` as the tolerance so entities can always escape the snap zone after one step, while still guaranteeing they never skip past a cell center.

### 3. Ghost AI
Each ghost uses a simple targeting system:
- 80% of the time: moves toward Pac-Man's current tile (greedy)
- 20% of the time: moves randomly
- In frightened mode: moves fully randomly
- When eaten: pathfinds back to the ghost house

### 4. Game Loop
The game runs via `requestAnimationFrame`. A state machine controls flow between screens:

```
title → ready → play → dying → dead → ready  (respawn loop)
                     ↘ win  → ready           (next level)
                     ↘ over                   (game over)
```

### 5. Bugs Fixed During Development

| Issue | Root Cause | Fix |
|---|---|---|
| Pac-Man frozen | `near()` tolerance (2.5) > speed (2.0), snapped back to center every frame | Pass `speed × 0.75` as tolerance |
| Arrow keys did nothing on title screen | Keys only buffered direction; user had to press SPACE first | Arrow keys now also trigger `initGame()` from title/game-over screen |
| Pac-Man didn't auto-move at game start | Initial `dx:0, dy:0` — Pac-Man sat still waiting for first input | Set `dx:-1` (left) on spawn, matching classic Pac-Man behavior |

---

## Plain HTML vs JSX — Why Plain HTML is the Better Choice Here

### What a JSX/React approach would require

```
project/
├── package.json
├── vite.config.js  (or webpack.config.js)
├── src/
│   ├── main.jsx
│   ├── components/
│   │   ├── GameCanvas.jsx
│   │   ├── HUD.jsx
│   │   └── Overlay.jsx
│   └── game/
│       ├── maze.js
│       ├── pac.js
│       └── ghosts.js
└── dist/           ← output after build
```

Before the game runs in a browser you need: `npm install`, a bundler, Babel transpilation, and a build command. Every deployment rebuilds the `dist/` folder.

### What this project uses

```
pacman_simugame/
└── index.html    ← the entire game, ~310 lines, open directly in any browser
```

### Direct comparison

| Concern | JSX / React | Plain HTML + Canvas |
|---|---|---|
| Setup | `npm install` + bundler config | None |
| Files | 10–50+ across `src/` and config | 1 |
| Build step | Required every deploy | None |
| Browser support | Requires transpilation | Native |
| Game loop | Fights React's render cycle | Direct `requestAnimationFrame` |
| Canvas access | `useRef` + `useEffect` workaround | `document.getElementById` |
| State | `useState`, `useReducer`, context | Plain `let` variables |
| Deployment | Upload compiled `dist/` | Upload one file |
| Debugging | Source maps + React DevTools | Browser devtools directly |
| Per-frame overhead | VDOM diffing on every render | Zero — direct canvas draw |

### Why React's model is wrong for a game loop

React's rendering model is designed for declarative UI: you describe *what* the screen should look like based on state, and React figures out *how* to update the DOM efficiently. This is excellent for forms, dashboards, and data-driven interfaces.

A game loop is the opposite: imperative, frame-by-frame, exact-pixel drawing. Every frame:
1. Clear the canvas
2. Draw the maze tile by tile
3. Draw every entity at its exact sub-pixel position
4. Move everything by exact pixel amounts based on speed

Wrapping this in React means fighting the abstraction at every step — `useRef` to escape the VDOM, `useEffect` to start the animation loop, careful dependency arrays to avoid re-creating the loop, and potential dropped frames from re-renders. You gain nothing from React's strengths and pay all of its complexity cost.

**Plain HTML + Canvas is the natural fit for game development:**
- Game state is just variables (`let pac`, `let gs`, `let mode`)
- The canvas context is a direct reference, no indirection
- The loop is a plain function called 60 times per second
- There is no framework to debug, upgrade, or fight

For a game, simplicity is performance and reliability. One file, zero dependencies, runs anywhere.

---

## Controls

| Key | Action |
|---|---|
| Arrow keys / WASD | Move Pac-Man |
| Space / Enter | Start game, continue after death |

Pac-Man starts moving left automatically when the game begins. Use arrow keys to change direction at any time.
