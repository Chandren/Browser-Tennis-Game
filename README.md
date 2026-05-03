# Browser-Tennis-Game
A minimalistic themed browser tennis game, can be played with a friend or with an adaptive AI which can learn from your moves after each game. 
 # TENNIS. — Nothing Edition

> *nothing to see here. just the game.*

A minimalist, browser-based 2D tennis game built with the **Nothing design language** — pure monochrome, dot-matrix typography, and clean geometric UI. No frameworks, no build step, no dependencies beyond two Google Fonts.

---

## Project Report

### Overview

**TENNIS.** is a self-contained, single-page web game that runs entirely in the browser using the HTML5 Canvas API. It was built to demonstrate how a polished, premium gaming experience can be achieved with vanilla HTML, CSS, and JavaScript — no libraries, no bundlers.

The aesthetic is a faithful interpretation of the [Nothing](https://nothing.tech) brand design language:
- `#0D0D0D` near-black backgrounds
- `#F5F5F5` near-white foreground elements
- `VT323` monospace font for score displays and headings
- `Inter` for UI labels and metadata
- Dot-grid patterns, subtle glassmorphism overlays, and micro-animations throughout

---

### Architecture

The project is **3 files**:

| File | Size | Role |
|---|---|---|
| `index.html` | 7 KB | DOM structure, 4 screens, semantic markup |
| `style.css` | 14 KB | Full design system — tokens, components, animations |
| `game.js` | 22 KB | Game engine, physics, AI, persistence |

**No build step. Open `index.html` in a browser and it runs.**

---

### Screen Flow

```
[START SCREEN]
     │
     ├── Player vs Player → [GAME SCREEN] ──────────────────┐
     └── Player vs AI    → [GAME SCREEN]                    │
                                │                            │
                            ESC pressed                  Score = 7
                                │                            │
                          [PAUSE OVERLAY]            [END SCREEN]
                                │                            │
                    Resume / Restart / Menu        Play Again / Menu
```

---

### Game Engine (`game.js`)

#### Physics

- Canvas dimensions: **1200 × 580** (scales responsively via CSS)
- Ball starts at centre with randomised direction each serve
- **Angled shots**: hit offset from the paddle centre maps linearly to `vy`, range `±8 px/frame`
- **Speed ramp**: each successful rally increases `|vx|` by 6%, capped at `16 px/frame` (base: `7`)
- Paddles can move both **vertically and horizontally** within defined lane boundaries:
  - P1 lane: `x ∈ [20, 68]`
  - P2 lane: `x ∈ [W−92, W−32]`
- Wall bounce: top/bottom edges reflect `vy`; left/right edges score a point and reset the ball
- **Camera shake**: 12-frame random translate on every goal scored

#### Rendering Pipeline (per frame)

1. Fill background `#181818`
2. Draw court lines (centre dash, baselines, horizontal midline, boundary edges)
3. Draw ball trail — 5-frame ghost with fading alpha
4. Draw ball — radial glow halo + solid `#F5F5F5` core
5. Draw paddles — rounded rects, white hit-glow that decays over 12 frames

#### AI System (PvE Mode) — 3 Adaptive Phases

The AI learns the player's shot tendencies across matches and sessions via `localStorage`.

| Phase | Shots Played | Behaviour |
|---|---|---|
| 0 — Reactive | 0 – 8 | Tracks the live ball Y position with damped smoothing (feels imperfect/human) |
| 1 — Adaptive | 9 – 25 | Blends ball trajectory prediction with player shot-tendency bias |
| 2 — Predictive | 26+ | Heavily pre-positions at the player's statistically favoured zone |

**Trajectory prediction**: when the ball travels toward the AI, it calculates time-to-arrival and simulates wall bounces using modular arithmetic to find where the ball will land.

**Tendency bias**: the AI records the last 20 `ball.y` positions when the player hits. It uses the rolling average as a "bias target" to counteract predictable player patterns.

**Adaptation factor** `= min(totalShots / 30, 0.75)` — grows across sessions, never resets unless the player clears `localStorage`.

**AI Speed** scales per phase and with total accumulated shots: `5.8 + phase × 0.5 + min(totalShots / 60, 1.5)` px/frame.

---

### Persistence (`localStorage`)

Two keys are maintained:

| Key | Contents |
|---|---|
| `tennis_hs_v1` | Top-5 PvE high scores (score, rallies, AI level, date) |
| `tennis_ai_stats_v1` | Accumulated AI learning data (shot counts, recency buffer, win rates) |

High scores are **sorted** by player score descending, then AI score ascending. New entries are highlighted with a slide-in animation.

---

### Design System (`style.css`)

Built as a token-first design system using CSS custom properties:

```css
--bg:          #0D0D0D   /* Page background          */
--surface:     #161616   /* Card / panel background  */
--surface-2:   #1E1E1E   /* Hover state              */
--border:      #2A2A2A   /* Subtle dividers          */
--border-2:    #383838   /* Prominent borders        */
--white:       #F5F5F5   /* Primary foreground       */
--grey-light:  #D4D4D4   /* Secondary text           */
--grey-mid:    #A3A3A3   /* Tertiary text            */
--grey-dark:   #6B6B6B   /* Labels, hints            */
```

**Animations included:**
- `cardIn` — overlay cards scale + fade in on show
- `scoreFlash` — score digits pulse on goal
- `hsRowIn` — new high score row slides in
- `shake` — CSS keyframe fallback (canvas shake done in JS)

---

## Features

- **Two game modes**: Local PvP (WASD vs Arrow Keys) and Solo PvE (Arrow Keys vs adaptive AI)
- **Angled physics**: ball angle on return depends on where it hits the paddle
- **Speed escalation**: ball gets faster every rally, capped at a max velocity
- **Horizontal paddle movement**: add depth to positioning strategy
- **3-phase adaptive AI** that learns your shot patterns over time and across sessions
- **Persistent AI memory** via localStorage — the AI gets harder every time you play
- **High score leaderboard** (top 5 PvE scores, persisted across sessions)
- **Live AI HUD** — adaptation %, shot tendency label, current rally counter
- **Pause menu** (ESC) with resume, restart, and main menu options
- **End screen** with full match stats and post-match high score highlight
- **Camera shake** on every scored point
- **Ball trail** effect for motion feel
- **Paddle hit glow** on successful shots
- **Dot-grid animated background** on start screen
- **Responsive canvas** — scales to fit any screen width

---

## Controls

| Action | PvP — Player 1 | PvP — Player 2 | PvE — Player |
|---|---|---|---|
| Move Up | `W` | `↑` | `↑` |
| Move Down | `S` | `↓` | `↓` |
| Move Left | `A` | `←` | `←` |
| Move Right | `D` | `→` | `→` |
| Pause / Unpause | `ESC` | `ESC` | `ESC` |

---

## Running the Game

No installation required.

```bash
# Option 1 — just open it
open index.html

# Option 2 — serve locally (avoids any browser file:// restrictions)
npx serve .
# or
python3 -m http.server 8080
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Structure | HTML5, semantic markup |
| Styling | Vanilla CSS, CSS Custom Properties |
| Logic | Vanilla JavaScript (ES2020, strict mode) |
| Rendering | HTML5 Canvas 2D API (`requestAnimationFrame` loop) |
| Fonts | Google Fonts — VT323, Inter |
| Persistence | Web Storage API (`localStorage`) |
| Build tooling | **None** |

---

## File Structure

```
Tennis game/
├── index.html   — 4-screen SPA (Start, Game, Pause overlay, End overlay)
├── style.css    — Design tokens, all component styles, animations
├── game.js      — Game engine, AI, physics, persistence
└── README.md    — This file
```

---

## Scoring

- First to **7 points** wins the match
- In PvE mode, your final score is recorded to the leaderboard
- High scores track: final score, total rallies, AI adaptation level at match end, and date

---

*v1.0 · nothing edition · 2026*
