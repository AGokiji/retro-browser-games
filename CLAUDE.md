# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ВАЖНОЕ ПРАВИЛО — Обязательно при каждом изменении

**После каждого изменения в проекте:**
1. Обновить этот файл `CLAUDE.md` (добавить/изменить описание новых фич, архитектуры, управления)
2. Сделать `git add -A`
3. Сделать `git commit -m "..."`
4. Сделать `git push`

Репозиторий: **https://github.com/AGokiji/retro-browser-games**

---

## Running the Games

Open any HTML file directly in a browser — no build step, server, or dependencies required:

```bash
open shooter.html
open tictactoe.html
```

---

## Repository Structure

| File | Description |
|---|---|
| `shooter.html` | Retro top-down shooter — main game |
| `tictactoe.html` | Tic-Tac-Toe with PvP and AI modes |
| `cycleforge.html` | CycleForge — Cycling Coach Dashboard (React 18 CDN + Babel standalone) |
| `CLAUDE.md` | This file — living documentation, always kept up to date |

Each game is a **single self-contained HTML file** with inline CSS and JS. No build tools, no package.json. External dependency: Google Fonts CDN (`Press Start 2P` font in shooter.html).

---

## Shooter Game Architecture (`shooter.html`)

Canvas: 800×600. All logic in one `<script>` block, sections marked with `// ──` comments.

### Global State
- `gameState` — FSM: `'menu'` → `'playing'` ↔ `'wavebreak'` ↔ `'upgrade'` → `'gameover'`
- `wave` — current wave number
- `score`, `bestScore` — score (best saved in `localStorage`)
- `spawnQueue[]` — pending enemy spawns for current wave
- `upgradeChoices[]` — 3 upgrade options shown on level-up

### Player Object
Flat object with all stats: `hp`, `maxHp`, `speed`, `xp`, `xpNext`, `level`, `dmgMult`, `fireRateMult`, `pierce`, `regen`, `shield`, `weapon` (0–4), parallel arrays `ammo[]`/`maxAmmo[]`/`fireRate[]`/`baseDmg[]`/`reloadTime[]`, buff timers `buffSpd`/`buffDmg`.

### Collections (arrays, mutated each frame)
`bullets[]`, `enemies[]`, `particles[]`, `pickups[]`, `flashes[]`, `lasers[]`

### Game Loop
`requestAnimationFrame(update)` → `update(dt)` (logic) → `draw(dt)` (render). Delta time capped at 50ms.

### Wave & Boss Logic
- `buildWaveQueue(w)` → returns array of `{type, delay}` spawn entries
- Wave `% 10 === 0` → miniboss only
- Wave `% 25 === 0` → boss only (3 phases at 66%/33% HP, stored in `e.phase`)
- `checkWaveClear()` → transitions to `'wavebreak'` when enemies + spawnQueue are empty
- After 3s break → `startNextWave()` increments wave and rebuilds spawn queue

### Weapons (player.weapon index)
| Index | Name | Special |
|---|---|---|
| 0 | Pistol | Default |
| 1 | Shotgun | 5-bullet spread |
| 2 | Auto | Fast fire rate |
| 3 | Rocket | AOE explosion (r=80) |
| 4 | Laser | Instant raycast, visual via `lasers[]` |

Unlocked at player levels: 1, 5, 10, 20, 30.

### Upgrade System
`offerUpgrades()` → pauses game, sets `gameState='upgrade'`, picks 3 random entries from `ALL_UPGRADES[]`. Each upgrade is `{id, label, apply()}`. Player presses 1/2/3.

### Enemy Types
`grunt`, `runner`, `tank`, `shooter` (ranged), `miniboss` (dash special), `boss` (ring bullets / spawn minions / dash, 3 phases).

### Pickups (dropped on kill, 25% chance)
`hp` (+30 HP), `spd` (speed buff 5s), `dmg` (damage ×2 for 5s), `shield` (absorb 1 hit), `xp` (+50 XP).

### Controls
| Key | Action |
|---|---|
| WASD / Arrows | Move |
| Mouse | Aim |
| LMB / Space | Shoot |
| R | Reload |
| ESC | Pause |
| Enter | Start / Retry |

---

## Tic-Tac-Toe (`tictactoe.html`)

Simple 3×3 board. Modes: PvP, Easy AI (30% minimax / 70% random), Hard AI (full minimax). Score persists in session. No localStorage.

---

## CycleForge Dashboard (`cycleforge.html`)

React 18 loaded via CDN + Babel standalone. No build step — open directly in browser.

### Tabs
| Tab | Content |
|---|---|
| Дашборд | KPI cards (CTL/ATL/TSB/TSS), PMC line chart, week quick view, macrocycle bar |
| Календарь | Full macrocycle list + current/previous week session rows |
| Тренировка | Session detail: stats grid, structure segments, back navigation |
| Зоны | FTP editor, 7-zone power table with computed watt ranges, FTP history, rider profile |
| Аналитика | PMC chart, HRV/recovery table, nutrition alerts, zone distribution, TSS bar |

### Key Data
- `MACRO[]` — 40 weekly entries (04авг 2025 → 04май 2026) with `label`, `tss`, `phase`, `planned`
- `pmcData[]` — 8 weeks CTL/ATL/TSB; `current:true` marks live week
- `SESSIONS[]` — 12 sessions (10–22 марта 2026), each with structure segments
- `PHASE_COLOR{}` — maps phase names to palette colours
- `FTP_ZONES[]` — 7 zones Z1–Z7 with labels, pct ranges, colours
- `FTP_HISTORY[]` — 5 test entries from Aug 2025 to Mar 2026

### Components
`Card`, `CardTitle`, `Badge`, `Alert`, `Stat`, `SessionRow` — UI primitives
`SVGLine(data, keys, h)` — polyline chart (PMC)
`SVGBar(data, h)` — bar chart (TSS macrocycle)
