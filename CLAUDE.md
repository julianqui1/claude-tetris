# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no `package.json`, no build, no lint, no tests. The README is in Spanish, and so is the in-game UI text (keep new UI strings in Spanish).

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`. Reload the browser to see changes.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, and `game.js` (all logic, one global script with `'use strict'`, no modules).

Key points in `game.js` that span multiple functions:

- **State** lives in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...), all reset by `init()`. The restart button just calls `init()`.
- **Piece encoding**: `PIECES[type]` is a square matrix whose non-zero cells equal the type index (1–7), which is also the index into `COLORS`. The board stores the same indices, so merging a piece preserves its color.
- **Piece lifecycle**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the spawn position collides. Hard drop, soft drop (when blocked) and the gravity tick in `loop()` all funnel through `lockPiece()`.
- **Game loop**: `requestAnimationFrame` `loop()` accumulates `dropAccum` and drops one row when it exceeds `dropInterval`. `animId` is cancelled on pause, game over and `init()`; resuming calls `loop()` directly after resetting `lastTime`. Any new code path that stops/starts the loop must keep `animId`/`lastTime` consistent to avoid double loops or a huge `dt` jump.
- **Rendering** is fully redrawn each frame in `draw()` (grid, board, ghost via `ghostY()`, current piece); `drawNext()` only runs from `spawn()`.
- **Scoring/speed** are computed in `clearLines()`: `LINE_SCORES[cleared] * level`; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`. Drop points (soft +1/row, hard +2/cell) are added in `softDrop()`/`hardDrop()`.
- **Canvas size coupling**: `COLS`/`ROWS`/`BLOCK` in `game.js` must match the `width`/`height` of `<canvas id="board">` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`, currently 300×600). The next-piece canvas is a fixed 120×120 using a hardcoded 30px block.
- **Rotation** (`tryRotate`) is clockwise only, with simple horizontal kicks `[0, -1, 1, -2, 2]` (not SRS).
