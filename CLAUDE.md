# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla JS Tetris. No dependencies, no build step, no package.json, no tests. Three files: `index.html`, `style.css`, `game.js`.

## Running

No install/build. Either open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no lint or test command in this repo.

## Architecture

All game logic lives in `game.js` (~300 lines) as top-level functions operating on module-level state variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — no classes, no modules.

- **Board**: `ROWS × COLS` matrix, each cell `0` (empty) or `1-7` (color index of the locked piece).
- **Pieces**: `PIECES` array of square matrices; `rotateCW` transposes+reverses rows to rotate. `tryRotate` applies wall-kick offsets `[0, -1, 1, -2, 2]`, using the first that doesn't `collide`.
- **Collision**: `collide(shape, ox, oy)` checks bounds and overlap against `board`.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, drops the piece one row when `dropAccum >= dropInterval`, otherwise calls `lockPiece()` (merge → clearLines → spawn).
- **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` × `level`; level increments every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece's landing row; drawn at `globalAlpha = 0.2`.
- **Rendering**: immediate-mode redraw each frame via 2D Canvas (`draw()` for the board/`next-canvas` for the preview) — no diffing/dirty-rect logic.
- **Input**: single `keydown` listener switches on `e.code` (arrows, `KeyX` rotate, `Space` hard drop, `KeyP` pause).

If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
