# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no `package.json`, no build step, no tests, no linter. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`, `npx serve .`) and visit `http://localhost:8000`.

## Architecture

All logic lives in `game.js`, a single script using module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...) and DOM lookups by id from `index.html`. Things that span files or aren't obvious:

- `index.html` element ids (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) are fetched at the top of `game.js`; renaming one breaks the script.
- Canvas size is hard-coded in `index.html` (`300x600` = `COLS*BLOCK` x `ROWS*BLOCK`). Changing `COLS`/`ROWS`/`BLOCK` requires updating the canvas attributes. The next-piece canvas is 120x120 (4x4 cells of 30px), with `NB = 30` hard-coded in `drawNext`.
- Board cells and piece matrices store color indices 1-7, which index both `COLORS` and `PIECES` (index 0 / `null` = empty). The same overlay element serves both PAUSA and GAME OVER; `endGame`/`togglePause` set its title text.
- Game loop: `loop` (requestAnimationFrame) accumulates `dropAccum` and gravity-drops when it exceeds `dropInterval`. Pause/game over cancel the frame via `animId`; `init()` (also the restart handler) resets all state and restarts the loop.
- Piece lifecycle: `lockPiece` = `merge` -> `clearLines` (updates lines/score/level/`dropInterval`) -> `spawn`. `spawn` promotes `next` and triggers `endGame` if the new piece collides immediately.
- Rotation is `rotateCW` plus simple horizontal kicks `[0,-1,1,-2,2]` in `tryRotate`; there is no SRS table.
- Scoring: `LINE_SCORES[cleared] * level`; soft drop +1 per row, hard drop +2 per row.
