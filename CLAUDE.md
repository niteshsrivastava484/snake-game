# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser Snake game. It is a single self-contained file, `index.html`, with inline CSS and JS and no dependencies. There is no build step, package manager, linter, or test suite.

Snakes & Ladders used to live here in `snakes-and-ladders/`. It now has its own repo, `niteshsrivastava484/snakes-and-ladders`. Don't add it back.

## Run / check / deploy

- **Run locally:** open `index.html` in a browser (`start "" index.html` on Windows).
- **Syntax check** (the only automated check available; needs Node):
  ```bash
  sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
  ```
- **Deploy:** push to `main`. GitHub Pages serves the repo root at https://niteshsrivastava484.github.io/snake-game/ and usually updates within 1–2 minutes. The file must stay named `index.html`, or the root URL returns 404.

## Git identity (personal repo; do not use FICO credentials)

- Remote: `git@github.com:niteshsrivastava484/snake-game.git`. Push directly to `main`; there is no PR flow.
- Repo-local config already set: `core.sshCommand` pointing to `~/.ssh/id_ed25519_personal` and `user.email` set to the personal Gmail. Keep both. The global git identity and default SSH key belong to a FICO enterprise account, which can't push here.
- For GitHub API actions (repo settings, Pages), use the `github-personal` MCP. The `github` MCP is signed in to the FICO account.
- `logs/` is tool output, not part of the game. It's listed in `.gitignore`.

## Architecture (all inside one IIFE in `index.html`)

- **State machine:** `state` is `idle | running | paused | over`. `start()`, `togglePause()` and `gameOver()` are the only places that change it, and they also start and stop the `setInterval(tick, speed)` timer. The overlay `<div>` is reused for the start, pause and game-over screens through `showOverlay()`.
- **Game loop:** `tick()` advances one cell per interval. Input never sets `dir` directly. `turn()` pushes to `dirQueue` (max 3, and it rejects reversals or repeats of the last queued direction), and `tick()` takes at most one entry per tick. This is what stops two fast key presses from reversing the snake into itself. The collision check leaves out the tail cell unless the snake is eating, so moving into the cell the tail is just leaving is allowed.
- **Rendering:** `draw()` repaints the whole 25×25 canvas each tick (`GRID = 25`, `CELL = canvas.width / GRID`). It draws the checkerboard, then the food, then the snake with a lightness gradient from head to tail, then eyes oriented to `dir`.
- **Theming:** the snake color is a single `hue` value. `applyColor()` writes it to the CSS variables `--accent` and `--accent-light`, which the title, button and credit link use. The canvas reads `hue` directly. To add a color, add an `<option value="<hue>">` to `#color`.
- **Persistence:** the browser's `localStorage` stores `snakeBest` (high score) and `snakeHue` (chosen color).
- **Input:** arrow keys or WASD, Space or P to pause, Enter to start, and touch swipes on the canvas (more than 20px). Changing the speed `<select>` restarts the interval if the game is running.
