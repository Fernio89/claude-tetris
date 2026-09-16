# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running and verifying

No build, no dependencies, no test suite, no linter — there is nothing to install and no npm
scripts. The whole app is `index.html` + `style.css` + `game.js`.

```bash
start index.html                  # Windows: open directly
python3 -m http.server 8000       # or serve statically, then open http://localhost:8000
```

Changes are verified by loading the page and playing: check the browser console for errors, then
exercise the affected path (spawn, rotate against a wall, line clear, level-up, pause, game over,
restart). There is no automated check that will catch a regression for you.

## Architecture

`game.js` is the entire game: module-level mutable state (`game.js:43`) plus a
`requestAnimationFrame` accumulator loop. `init()` is both the boot path and the restart path — it
rebuilds the board, resets every state variable, cancels the in-flight rAF, and is bound directly to
the restart button. Frame flow: `loop()` accumulates `dt`, drops the piece one row when
`dropAccum >= dropInterval`, otherwise `lockPiece()` → `merge()` → `clearLines()` → `spawn()`; a
collision at spawn ends the game.

### Invariants worth knowing before editing

- **Board cells hold color indices, not booleans.** Each piece matrix in `PIECES` is filled with its
  own type number, `merge()` copies those numbers into `board`, and `drawBlock()` indexes `COLORS`
  with them. `COLORS` and `PIECES` are index-aligned and both begin with a `null` at index 0,
  because `0` means "empty cell". Adding a piece means appending to *both* arrays.
- **Canvas dimensions live in the HTML, board dimensions in the JS.** `#board` is `300x600` in
  `index.html`; `COLS`/`ROWS`/`BLOCK` are in `game.js`. Changing `COLS`, `ROWS` or `BLOCK` requires
  editing the canvas attributes to match (`COLS * BLOCK` x `ROWS * BLOCK`). The preview is the same
  deal: `#next-canvas` is `120x120` and `drawNext()` centers the shape in a fixed 4x4 grid at 30px,
  so a piece wider than 4 cells would overflow it.
- **DOM ids are resolved at script load**, at the top of `game.js`, since the script tag sits at the
  end of `<body>` with no `defer`. Renaming an id in `index.html` fails at load time, far from the
  code that uses it.
- **Rotation is transpose-then-reverse** (`rotateCW`) with a simple kick list `[0,-1,1,-2,2]` tried
  in order (`tryRotate`) — not SRS. There is no wall-kick table, no O-piece special case, and no
  lock delay.
- **Resuming from pause resets `lastTime`**, otherwise the first `dt` after the pause would be the
  whole paused duration. Any new code that stops and restarts the loop must do the same. One
  overlay element is shared by the PAUSA and GAME OVER states, switched by its text content and the
  `hidden` class.
- **Difficulty is derived, not stored**: `level = floor(lines / 10) + 1` and
  `dropInterval = max(100, 1000 - (level - 1) * 90)`, both recomputed in `clearLines()`.

## Conventions

User-facing text (overlay strings, the on-screen control list, README) is in Spanish; identifiers
and code comments are in English. `game.js` is `'use strict'` plain ES6+ — no modules, no imports,
no transpilation; anything added must run as-is in the browser.
