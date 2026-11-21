# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

An Asteroids-style arcade game built with plain HTML5 Canvas and vanilla JavaScript. There are no dependencies, no bundler, no build step, and no test suite or linter. The whole game lives in `game.js`, which `index.html` loads with a plain `<script>` tag (not an ES module). UI text and code comments are in Spanish, and new UI strings should be too.

## Running

Open `index.html` directly in a browser, or serve the directory:

```bash
npx serve .   # then visit http://localhost:3000
```

The README links to a demo hosted on GitHub Pages, so `index.html` must keep working as a static page at the repo root.

## Architecture (`game.js`)

- **Fixed logical canvas**: `W = 800`, `H = 600` are hardcoded and must match the `<canvas>` size in `index.html`.
- **Global mutable state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` and `deadTimer` are module-level `let`s. `initGame()` resets everything, and `nextLevel()` respawns `3 + level` large asteroids.
- **State machine**: `state` is `'playing' | 'dead' | 'gameover'`. `update(dt)` branches on it at the top. In `'dead'`, asteroids and particles keep moving until `deadTimer` runs out and the ship respawns. In `'gameover'`, Space restarts the game.
- **Entity convention**: every entity class (`Bullet`, `Asteroid`, `Ship`, `Particle`) has `update(dt)`, `draw()` and a `dead` flag. The main update loop marks entities dead, then removes them by reassigning the array with `.filter(x => !x.dead)`. Follow this pattern for any new entity type.
- **Toroidal space**: positions are wrapped with `wrap(v, max)`. Particles are the exception and don't wrap.
- **Asteroid sizes**: `size` is 3 (large), 2 (medium) or 1 (small). The `RADII`, `SPEEDS` and `POINTS` arrays are indexed by `size` (index 0 is unused). `split()` returns two asteroids one size smaller.
- **Collisions**: circle-distance checks with `dist()`. Ship-vs-asteroid uses `a.radius * 0.82` to make hits more forgiving, and is skipped while `ship.invincible > 0`.
- **Input**: `keys[code]` holds whether a key is currently down, for continuous actions like rotating and thrusting. `pressed(code)` is an edge trigger that consumes `justPressed[code]`, for one-shot actions like shooting and restarting. Shooting also has a 0.2s cooldown, `shootCooldown`.
- **Loop**: `requestAnimationFrame` drives the game, with `dt` in seconds capped at 0.05. Movement constants are in px/s, so scale any per-frame change by `dt`.
