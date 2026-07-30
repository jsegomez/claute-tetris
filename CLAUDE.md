# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Vanilla Tetris implementation using HTML5 Canvas, CSS, and plain JavaScript (ES6+). No dependencies, no build step, no package.json, no test suite.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test command — the three files are used as-is.

## Architecture

The project is three files that cooperate directly (no modules/bundler):

- `index.html` — DOM structure: main `<canvas id="board">` (300×600, 10×20 grid at `BLOCK=30`px), a `<canvas id="next-canvas">` for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and the pause/game-over `#overlay`.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, in a single top-to-bottom script (no classes, mutable module-level state).

### Core state and flow (`game.js`)

Module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) hold all game state — there's no state container object.

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: `PIECES` are defined as square matrices; rotation is done via matrix transpose+reverse (`rotateCW`), not precomputed rotation states.
- **Collision** (`collide`): checks board bounds and existing locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time and drops the piece one row when `dropAccum >= dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows and unshifts empty ones at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × current level; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Spawning a piece that immediately collides (in `spawn()`) triggers `endGame()`.

Keyboard input is handled by a single `keydown` listener at the bottom of `game.js` (arrows + `X` to rotate, `Space` for hard drop, `P` to pause); it's a flat switch, not an input-abstraction layer.

## Tunable constants (`game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
