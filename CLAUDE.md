# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Two browser games, each a single self-contained HTML file with inline CSS and JS and no dependencies. There is no build step, package manager, linter, or test suite.

- `index.html`: Snake, served at the site root
- `snakes-and-ladders/index.html`: Snakes & Ladders, served at `/snakes-and-ladders/`

The two pages link to each other through their credit lines, using relative links (`snakes-and-ladders/` and `../`). Keep those links working if you move files.

## Run / check / deploy

- **Run locally:** open `index.html` in a browser (`start "" index.html` on Windows).
- **Syntax check** (the only automated check available; needs Node). Run it against whichever file you changed:
  ```bash
  f=index.html   # or snakes-and-ladders/index.html
  sed -n '/<script>/,/<\/script>/p' "$f" | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
  ```
- **Deploy:** push to `main`. GitHub Pages serves the repo root at https://niteshsrivastava484.github.io/snake-game/ and usually updates within 1–2 minutes. The file must stay named `index.html`, or the root URL returns 404.

## Git identity (personal repo; do not use FICO credentials)

- Remote: `git@github.com:niteshsrivastava484/snake-game.git`. Push directly to `main`; there is no PR flow.
- Repo-local config already set: `core.sshCommand` pointing to `~/.ssh/id_ed25519_personal` and `user.email` set to the personal Gmail. Keep both. The global git identity and default SSH key belong to a FICO enterprise account, which can't push here.
- `logs/` is tool output, not part of the games. It's listed in `.gitignore`.

## Snake architecture (all inside one IIFE in `index.html`)

- **State machine:** `state` is `idle | running | paused | over`. `start()`, `togglePause()` and `gameOver()` are the only places that change it, and they also start and stop the `setInterval(tick, speed)` timer. The overlay `<div>` is reused for the start, pause and game-over screens through `showOverlay()`.
- **Game loop:** `tick()` advances one cell per interval. Input never sets `dir` directly. `turn()` pushes to `dirQueue` (max 3, and it rejects reversals or repeats of the last queued direction), and `tick()` takes at most one entry per tick. This is what stops two fast key presses from reversing the snake into itself. The collision check leaves out the tail cell unless the snake is eating, so moving into the cell the tail is just leaving is allowed.
- **Rendering:** `draw()` repaints the whole 25×25 canvas each tick (`GRID = 25`, `CELL = canvas.width / GRID`). It draws the checkerboard, then the food, then the snake with a lightness gradient from head to tail, then eyes oriented to `dir`.
- **Theming:** the snake color is a single `hue` value. `applyColor()` writes it to the CSS variables `--accent` and `--accent-light`, which the title, button and credit link use. The canvas reads `hue` directly. To add a color, add an `<option value="<hue>">` to `#color`.
- **Persistence:** the browser's `localStorage` stores `snakeBest` (high score) and `snakeHue` (chosen color).
- **Input:** arrow keys or WASD, Space or P to pause, Enter to start, and touch swipes on the canvas (more than 20px). Changing the speed `<select>` restarts the interval if the game is running.

## Snakes & Ladders architecture (`snakes-and-ladders/index.html`)

- **Board:** an SVG with `viewBox` 0–500, drawn once at load. `center(n)` maps square `n` (1–100) to pixel coordinates in a zigzag: square 1 is bottom-left, and the row direction alternates every 10 squares. Every placement goes through `center()`.
- **Snakes and ladders:** these are data only, in the `SNAKES` and `LADDERS` maps (`{ landOn: endUp }`). Snake bodies are generated sine curves running from head to tail. Edit the maps and the drawing follows. A square must not appear as a key in both maps.
- **Turn flow:** `roll()` is async: it animates the die, steps the token one square at a time, then applies a snake or ladder. `busy` blocks input while a move is animating. `gameId` goes up on every `newGame()`, and each `await` checks it, so starting a new game mid-move stops the old move cleanly.
- **Rules:** a roll that would go past 100 doesn't move the token; you have to land on 100 exactly. A 6 gives another roll. In "You vs Computer" mode, `renderTurn()` triggers the computer's roll after a short delay.
- **Tokens:** each token is an SVG `<g>` moved with a CSS `transform`. They use the `slide` class for the slower snake/ladder animation. `OFFSETS` gives each player a fixed spot inside a square, so tokens on the same square don't overlap.
- **Persistence:** the browser's `localStorage` stores `slMode`, the selected player mode.
