# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js`, `index.html`, or `style.css` take effect on page reload with no compilation step.

## Architecture

Everything lives in three files that cooperate directly via DOM element IDs (no modules, no imports):

- `index.html` — DOM structure: `<canvas id="board">` (300×600, the main play field), `<canvas id="next-canvas">` (next-piece preview), HUD spans (`score`, `lines`, `level`), and a shared `overlay` div used for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, in a single top-to-bottom script (no classes, no build-time modules).

### Core model

- The board is a `ROWS × COLS` (20×10) matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type locked there.
- Pieces (`PIECES`) are defined as square matrices of color indices; rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table of pre-rotated states.
- `collide(shape, ox, oy)` is the single collision primitive — checked before every move, rotation, and drop.
- `tryRotate()` implements basic wall kicks: after rotating, it tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.

### Game loop

`loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time (`dropAccum`) and advancing the piece one row once `dropInterval` is exceeded, otherwise calling `lockPiece()` on collision. `lockPiece()` merges the current piece into the board, clears completed lines, and spawns the next piece. All keyboard input (`keydown` listener at the bottom of `game.js`) mutates `current` synchronously between frames; there is no input queue.

### Scoring/leveling coupling

`clearLines()` ties three concerns together in one place: line-clear scoring (`LINE_SCORES` table × `level`), level-up (every 10 lines), and drop-speed recalculation (`dropInterval = max(100, 1000 - (level-1)*90)`). Any change to leveling or speed curves should go through this function, not the loop.

### Rendering

`draw()` redraws the whole board every frame in a fixed order: grid lines → locked board cells → ghost piece (`ghostY()` projects straight down, drawn at `globalAlpha = 0.2`) → current piece. `drawNext()` renders the preview canvas independently using the same `drawBlock()` primitive at a different block size (`NB = 30` vs `BLOCK = 30`).

### State

All mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) is declared as module-level `let`s and reset in `init()`. There's no state container object — functions read/write these globals directly.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
