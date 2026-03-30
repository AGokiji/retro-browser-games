# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Репозиторий: **https://github.com/AGokiji/retro-browser-games**

После каждого изменения: обновить `CLAUDE.md`, `git add -A`, `git commit`, `git push`.

---

## Running the Projects

### HTML games (no build step)
```bash
open shooter.html
open tictactoe.html
```

### Claude Artifact Runner (Vite + React)
```bash
cd claude-artifact-runner
npm install      # first time only
npm run dev      # dev server at http://localhost:5173
npm run build    # production build
npm run preview  # preview production build locally
npm run lint     # ESLint
```

---

## Repository Structure

| File / Dir | Description |
|---|---|
| `shooter.html` | Retro top-down shooter — single self-contained HTML/JS/CSS file |
| `tictactoe.html` | Tic-Tac-Toe with PvP and AI modes — single self-contained HTML file |
| `CycleForge_4.jsx` | CycleForge cycling coach dashboard — JSX artifact for the artifact runner |
| `claude-artifact-runner/` | Vite + React 19 + Tailwind + Shadcn UI environment for running JSX artifacts |
| `CLAUDE.md` | This file — living documentation, always kept up to date |

HTML games are self-contained with inline CSS/JS. External dependency: Google Fonts CDN (`Press Start 2P` in shooter.html).

`claude-artifact-runner/` is a **separate git repository** (its own `.git`). Git commands run from the project root do not affect it.

### Claude Artifact Runner workflow
Paste any Claude-generated JSX into `claude-artifact-runner/src/Artifact.jsx` and the dev server hot-reloads it. All Shadcn UI components (`@/components/ui/…`), Lucide icons, Recharts, and Tailwind are pre-installed. `CycleForge_4.jsx` is the current dashboard artifact — copy its contents into `Artifact.jsx` to run it.

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

## CycleForge Dashboard (`CycleForge_4.jsx`)

React 19 component. Run it by pasting into `claude-artifact-runner/src/Artifact.jsx` and starting `npm run dev`.

### Tabs
| Tab | Content |
|---|---|
| Дашборд | KPI cards (CTL/ATL/TSB/TSS), PMC line chart, week quick view, macrocycle bar |
| Календарь | Full macrocycle list + current/previous week session rows |
| Тренировка | Session detail: stats grid, structure segments, back navigation |
| Зоны | FTP editor, 6-zone power table with computed watt ranges, FTP history, rider profile |
| Аналитика | PMC chart, HRV/recovery table, nutrition alerts, zone distribution, TSS bar |

### Key Data (`CycleForge_4.jsx`)
- `ATHLETE_DEFAULTS` — rider profile: FTP, weight, HR max, LTHR, FTP history
- `calcZones(ftp)` — derives 6 power zones (Z1–Z6) dynamically from FTP; stored in state via `useState`
- `BLOCKS[]` — 7 training blocks (Base → Race A, Aug 2025 → May 2026) with `id`, `name`, `start/end`, `color`, `active`
- `WEEK_PLAN[]` — current week sessions (Mon–Sun) with type, title, TSS, zone

### Components (`CycleForge_4.jsx`)
Uses Shadcn UI primitives (`Card`, `Badge`, `Button`, `Tabs`, etc.) and Recharts (`BarChart`, `LineChart`, `AreaChart`, `PieChart`, `RadarChart`) — all pre-installed in the artifact runner.
