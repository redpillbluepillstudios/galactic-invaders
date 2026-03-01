# Brownfield Analysis
This document captures the technical audit of an existing codebase performed by the **AGENT**. It serves as the foundation for generating the PRD and Plan documents for a brownfield project.

## Project Overview
Galactic Invaders is a retro-style 2D space shooter browser game targeting retro gaming enthusiasts and nostalgic players. Built entirely with vanilla JavaScript and PixiJS, it delivers a polished arcade experience with zero setup — playable instantly in any browser. The game features 8-directional player movement, multiple enemy types, power-ups, progressive difficulty, and a city defense mechanic, all wrapped in a neon-glow retro arcade aesthetic.

**Current Version:** 0.6.4 (Help System)
**Live URL:** https://galacticinvaders.icodewith.ai/
**Repository:** https://github.com/bymarcelolewin/galactic-invaders

## Tech Stack
| Technology | Purpose |
|-----------|---------|
| HTML5 | Page structure and semantic markup |
| CSS3 | Retro arcade styling, responsive layout, glow effects |
| JavaScript (ES6+) | All game logic, vanilla with no frameworks |
| PixiJS v6.5.8 (CDN) | 2D rendering engine for game canvas |
| PixiJS Glow Filter v4 (CDN) | Visual glow effects on explosions and player |
| Marked.js (CDN) | Markdown parser for in-game release notes display |
| Google Fonts (Press Start 2P) | Retro pixel-style typography |
| HTML5 Audio API | Sound effects (shoot, explosion, phantom alien) |

**No build tools, no bundler, no transpiler.** All external libraries loaded via CDN. No npm dependencies (package.json is metadata only).

## Project Structure
```
galactic-invaders/
├── assets/                  # Images (PNG), sounds (WAV), and CSS
│   ├── style.css            # All styling — retro arcade aesthetic
│   ├── favicon.ico          # Browser tab icon
│   ├── galactic-invaders.png # Screenshot for README
│   ├── shoot.wav            # Shooting sound effect
│   ├── explosion.wav        # Explosion sound effect
│   └── phantom.wav          # Phantom alien sound effect
├── js/                      # JavaScript source files
│   ├── game.js              # Core game logic (~1370 lines)
│   ├── devMode.js           # Developer mode popup (Shift+6)
│   ├── releaseNotesMode.js  # Release notes popup (Shift+5)
│   └── helpMode.js          # Help/controls popup (H key)
├── rules/                   # Game configuration
│   └── game_rules.json      # All game balance parameters
├── index.html               # Single-page entry point
├── package.json             # Minimal metadata (name, version, description)
├── release_notes.md         # Version history document
├── README.md                # Project documentation
└── .gitignore               # Git exclusions
```

## Key Files
| File | Description |
|------|-------------|
| `index.html` | Single entry point. Loads fonts, CDN libraries, game scripts. Contains arcade frame, HUD, game over overlay, and 3 popup modals. |
| `js/game.js` | Core game engine (~1370 lines). Handles PixiJS initialization, game loop, player movement, alien spawning, collision detection, power-ups, difficulty progression, explosions, HUD updates, and all game state. |
| `js/devMode.js` | Developer mode toggle (Shift+6). Displays game_rules.json content. Pauses game ticker. |
| `js/releaseNotesMode.js` | Release notes viewer (Shift+5). Fetches and renders markdown via Marked.js. Pauses game ticker. |
| `js/helpMode.js` | Help screen (H key). Displays hardcoded HTML with controls and objectives. Pauses game ticker. |
| `rules/game_rules.json` | Configuration-driven game balance. Point values, difficulty thresholds, spawn intervals, alien HP, power-up rules, building counts, phantom alien parameters. |
| `assets/style.css` | Complete styling. Retro arcade aesthetic with neon green borders, glow effects, responsive 4:3 aspect ratio container, HUD positioning, popup overlays. |

## Dependencies
All dependencies are loaded via CDN — no package manager required:

| Dependency | Version | Purpose |
|-----------|---------|---------|
| PixiJS | 6.5.8 | 2D WebGL/Canvas rendering engine for the game |
| PixiJS Glow Filter | 4.x | Glow visual effects on explosions and player ship |
| Marked.js | latest | Markdown-to-HTML parser for release notes display |
| Press Start 2P (Google Fonts) | — | Retro pixel font used throughout the UI |

## Architecture
**Single-page browser application** with no backend, no database, and no build process.

- **Rendering:** PixiJS manages a WebGL/Canvas element for all game graphics (player, aliens, bullets, explosions, starfield, city).
- **Game Loop:** PixiJS `app.ticker` runs at 60fps, handling all updates (movement, spawning, collision detection, animations).
- **State Management:** All game state stored in global JavaScript variables (score, lives, aliens array, bullets array, etc.).
- **Configuration:** Game balance parameters externalized to `rules/game_rules.json`, fetched at startup.
- **UI Modules:** Popup features (dev mode, release notes, help) separated into individual JS files that share a mutual exclusion pattern — only one popup can be open at a time, all pause the game ticker.
- **Input:** Single keyboard event listeners handle all controls. WASD mapped to arrow key equivalents.
- **Audio:** HTML5 Audio API with sound toggle (M key). Audio objects reused via `currentTime` reset.
- **Responsive Design:** CSS-based responsive container with 4:3 minimum aspect ratio. Title screen layout adapts to screen size.

## Data Model
No persistent data storage. All game state is in-memory JavaScript variables:

**Player State:** position (x, y), lives, score, nukes count, rapid fire status/timer, movement key states

**Alien Objects:**
```
{ x, y, vx, vy, alienWidth, alienHeight, hp, isTough, isPhantom, fromLeft }
```
- Normal aliens: 1 HP, 100 points
- Tough aliens: 3 HP, 300 points, purple color
- Phantom aliens: 3 HP, 500 bonus points, fast horizontal movement, unique sound

**Explosion Objects (pooled):**
```
{ x, y, gfx (PIXI.Graphics), age, maxAge, glowFilter }
```

**Building Objects:**
```
{ x, y, width, height, PIXI.Graphics with drawn windows }
```

**Game Progression:** difficulty level, spawn interval timer, phantom alien score tracking

## Existing Features

### Core Gameplay
- 8-directional player movement (Arrow keys / WASD)
- Shooting with cooldown system (Space bar)
- Rapid fire power-up — unlocked at 2000-point intervals, lasts 5 seconds
- Nuke weapon — screen-clear ability unlocked every 3000 points (Q key)
- Progressive difficulty — aliens spawn faster as score increases (every 750 points)
- Lives system — 3 starting lives, lost on alien-player collision
- City defense — random buildings (4-8) at bottom that aliens can destroy
- City base — decorative pixelated base that triggers game over when aliens reach it

### Enemy Types
- **Normal aliens** — Standard enemies, 1 HP, 100 points, 25% random spawn
- **Tough aliens** — Purple color, 3 HP, 300 points, 75% spawn chance
- **Phantom alien** — Rare special enemy at 2000-point intervals, fast horizontal movement, 3 HP, 500 bonus points, unique audio cue

### Visual Effects
- Scrolling parallax starfield (80 animated stars)
- Pixelated starburst explosions with glow filter and fade animation
- Object pooling for explosion graphics (performance optimization)
- Pulsating green glow on player ship during rapid fire
- HUD value pop animations (scale bounce on score/lives changes)
- Neon green arcade frame with CSS glow effects

### UI/UX
- Responsive title screen with game title, version, instructions, alien types
- In-game HUD showing score, lives, nukes, buildings count
- Game over screen with restart button
- Help popup (H key) — controls and objectives
- Release notes popup (Shift+5) — markdown-rendered version history
- Developer mode popup (Shift+6) — displays game_rules.json
- Popup mutual exclusion — only one popup open at a time
- Sound toggle (M key) — mute/unmute all audio
- Responsive design adapting to screen size (4:3 minimum aspect ratio)

## User Q&A

**AGENT Q1:** Who is the target audience for Galactic Invaders?
**USER A1:** Retro gaming enthusiasts and nostalgic players who grew up with 80s arcade games and enjoy the pixel art, neon glow, and classic shoot-em-up mechanics.

**AGENT Q2:** What problem does Galactic Invaders solve, or what need does it fulfill?
**USER A2:** It fills the gap between overly simple browser games and complex downloads — delivering a polished arcade experience with zero setup, playable instantly in any browser.

**AGENT Q3:** How would you define success for Galactic Invaders?
**USER A3:** Players had fun. If someone plays it and enjoys themselves, the game succeeded.

**AGENT Q4:** What do you see as the must-have features for the next milestone?
**USER A4:** Mobile/touch controls — making the game accessible on phones and tablets, which is a huge audience expansion.

**AGENT Q5:** Are there any constraints or risks I should know about?
**USER A5:** No constraints beyond what's already in place. Keep the current vanilla JS approach, no frameworks, no build tools, single-page architecture.

## Summary
Galactic Invaders is a polished, fully playable retro arcade space shooter at version 0.6.4. It targets retro gaming enthusiasts who want an instant, no-download arcade experience in the browser. Built with vanilla JavaScript and PixiJS on a single HTML page with no build process, it features 3 enemy types, power-ups (rapid fire, nukes), progressive difficulty, city defense, and a complete retro arcade aesthetic with neon glow effects and pixel fonts.

The game succeeds when players have fun — simple as that. The immediate priority for the next milestone is adding mobile/touch controls to expand accessibility to phones and tablets. Future nice-to-haves include high score persistence/leaderboards, boss fights, additional power-ups, and deeper visual/audio polish. The project will continue using vanilla JS with CDN dependencies, no frameworks, and no build tools.
