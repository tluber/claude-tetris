# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Vanilla Tetris implementation using HTML5 Canvas, CSS, and plain JavaScript (ES6+). No build process, no package manager, no external dependencies.

## Running the game

No install/build step. Either open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no tests, linter, or build/bundle commands in this repo.

## Architecture

Three files cooperate, all logic lives in `game.js` (~300 lines):

- **`index.html`** — DOM structure: main `<canvas id="board">` (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), a side panel with score/lines/level and a `<canvas id="next-canvas">` preview, and a hidden overlay div reused for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game logic:
  - **Board model**: a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece.
  - **Pieces**: defined as square matrices; rotation is computed via transpose + row-reverse (`rotateCW`).
  - **Collision detection** (`collide`): checks board bounds and overlap with locked cells.
  - **Wall kicks** (`tryRotate`): on rotation collision, tries shifting the piece ±1/±2 columns before giving up on the rotation.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time and drops the piece one row when `dropInterval` is exceeded.
  - **Line clearing** (`clearLines`): scans bottom-to-top; full rows are removed and empty rows inserted at the top.
  - **Scoring**: classic table `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current level; hard drop adds 2 pts/cell dropped, soft drop adds 1 pt/row.
  - **Level/speed**: level increases every 10 lines; drop speed is `max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece** (`ghostY`): projects where the current piece would land and draws it at `globalAlpha = 0.2`.

Control flow:

```
init() → createBoard() → next = randomPiece() → spawn() → requestAnimationFrame(loop)

loop(timestamp):
  accumulate dt → if dt ≥ dropInterval, drop the piece or call lockPiece()
  draw() (grid + board + ghost + current piece)
  requestAnimationFrame(loop)

keydown → move / rotate / soft-drop / hard-drop / pause
```

If a freshly spawned piece immediately collides (in `spawn()`), `endGame()` fires and the GAME OVER overlay is shown.

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
