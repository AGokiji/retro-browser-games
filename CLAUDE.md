# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open any HTML file directly in a browser — no build step, server, or dependencies required:

```bash
open shooter.html
open tictactoe.html
```

## Repository Structure

Each game is a **single self-contained HTML file** with inline CSS and JS — no external dependencies except the Google Fonts CDN (`Press Start 2P` font in shooter.html).

## Shooter Game Architecture (`shooter.html`)

All logic lives in one `<script>` block. Key sections (marked with `// ──` comments):

- **STATE** — global mutable state: `gameState`, `wave`, `score`, `spawnQueue`, `upgradeChoices`
- **PLAYER object** — all player stats, weapon slots, buff timers in one flat object
- **COLLECTIONS** — `bullets[]`, `enemies[]`, `particles[]`, `pickups[]`, `flashes[]`, `lasers[]`
- **Game loop** — `requestAnimationFrame(update)` calls `update(dt)` then `draw(dt)` every frame
- **`gameState` FSM** — `'menu'` → `'playing'` ↔ `'wavebreak'` ↔ `'upgrade'` → `'gameover'`

### Wave & Boss Logic
- `buildWaveQueue(w)` generates the enemy spawn list for wave `w`
- Wave `% 10 === 0` → miniboss only; wave `% 25 === 0` → boss only
- Boss has 3 phases tracked by `e.phase` (changes at 66%/33% HP)
- `checkWaveClear()` called each frame; transitions to `'wavebreak'` when `enemies` and `spawnQueue` are both empty

### Weapons
5 weapon slots on `player.weapon` (0–4): pistol, shotgun, auto, rocket, laser. Each has parallel arrays: `ammo[]`, `maxAmmo[]`, `fireRate[]`, `baseDmg[]`, `reloadTime[]`. Laser uses `fireLaser()` (instant raycast + `lasers[]` for visual); rocket uses AOE splash on bullet impact.

### Upgrade System
`offerUpgrades()` pauses the game and sets `gameState='upgrade'`. Player presses 1/2/3 to pick from `upgradeChoices[]`. Upgrades are defined in `ALL_UPGRADES[]` as `{id, label, apply()}` objects.

## GitHub

All changes must be committed and pushed to: **https://github.com/AGokiji/retro-browser-games**

```bash
git add <file>
git commit -m "description"
git push
```
