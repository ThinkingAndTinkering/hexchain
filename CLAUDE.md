# Hex Chain

## Project Overview
Browser-based mobile puzzle game. Single HTML file, no frameworks, no build step. Canvas 2D rendering for the hex grid, DOM for chrome/overlays.

## Architecture
- **Rendering**: Canvas 2D with `requestAnimationFrame` + dirty-flag system
- **Hex Grid**: Axial coordinates (q, r), pointy-top hexagons, grid radius 3 (37 cells)
- **Coordinate Math**: `axialToPixel` / `pixelToAxial` / `axialRound` for conversions
- **Gravity**: Hex cascade — cell-by-cell bottom-up pull. Each empty cell pulls from upper-left `(q, r-1)` or upper-right `(q+1, r-1)`, random tiebreak when both have tiles. Iterates until stable (max 10 passes). Diagonal settle animations with both X and Y offsets.
- **Input**: Pointer Events (pointerdown/move/up) for cross-device touch-drag path drawing
- **Animations**: Time-based system with Maps tracking clearing/settling/appearing/ripple states
- **Particles**: Sparkle effects on chain clears + ambient floating particles + confetti on new high scores
- **DPR-aware**: Canvas scales for retina displays

## Game Mechanics
- Draw paths through adjacent same-colored hex tiles (min chain: 3)
- Tiles clear along the path with staggered animation, gravity pulls tiles down diagonally (plinko-style), new tiles fill gaps
- **Freeze mechanic** (Classic only): Each tile has an `age` counter. After 8 moves without being cleared, tiles freeze (darkened overlay + snowflake, can't be used in chains). Countdown numbers (3, 2, 1) show on tiles approaching freeze in the tile's own color. Pulsing colored border on tiles 1 move from freezing.
- **Chain reward tiers (v2 — current, testable)**:
  - **4+ chain → Color Wipe**: Clears ALL remaining tiles of that color from the board with radial wave stagger, bonus points (50 per wiped tile), ripple effect, "COLOR WIPE!" float text, score shake
  - **5+ chain → Color Wipe + Same-Color Reset** (Classic only): All tiles of that color get age reset to 0 (frozen tiles thawed with ice-crack particles)
  - **6+ chain → Color Wipe + Global Reset** (Classic only): ALL tiles of ALL colors get age reset to 0 (frozen tiles thawed, "BOARD RESET!" float text)
- **Previous chain tiers (v1 — for revert)**: 5+ color wipe, 6+ global thaw to age 5, normal chains thaw+refresh same-color tiles within 3 moves of freezing
- **Manual reshuffle**: Players get 2 reshuffles per game (button in info bar with ↻ icon + count). When no valid chains exist, reshuffle button pulses and a persistent red "NO MORE MOVES" banner shows. If no reshuffles remain, game ends.
- **Classic mode**: Unlimited moves, 6 colors. Frozen tiles cause board lockup. No timer.
- **Sprint mode**: 45-second timer, 6 colors, NO freezing. Timer starts on first chain. Long chains earn bonus time: 4-chain +2s, 5-chain +4s, 6-chain +7s, 7+ chain +10s. Prominent timer display (28px, accent color).
- Chain tier feedback: Nice! (3) → Great! (4) → Amazing! (5) → Fantastic! (6) → INCREDIBLE! (7) → LEGENDARY! (8) → GODLIKE!! (10+)
- Deadlock detection via flood-fill connected components (excludes frozen tiles)
- **Game Over phrases**: Random selection from: "HEXED!", "HEX-AGAIN?", "SIX SIDES, ONE MORE TRY", "TOTAL HEX-LAPSE"

## Tile Colors (8 defined, 6 used in all modes)
Slate Blue (#5B8CB8), Sage (#5A9E7E), Ochre (#C8A855), Terracotta (#B85C5C), Plum (#8B6BAA), Copper (#C47850), Fern (#6AAA5A), Pewter (#A0A0A8). All have base/dark/light variants for gem gradient rendering.

## Scoring
3-chain: 150, 4: 400, 5: 850, 6: 1500, 7: 2500, 8: 4000, 9: 6500, 10: 10000, 10+: exponential (~×1.6 per additional tile). Color wipe bonus: 50 per wiped tile.
Target ranges: fast game 5-10K, good game 10-30K, great game high 5 digits.

## Storage
- localStorage for high scores per mode, tutorial seen flag, leaderboard (top 100 entries with timestamps, filtered by today/month/all-time for display)

## Visual Design
- Premium dark theme: `--bg-dark: #141419`, radial gradient background
- **Steel blue accent `#7EB8D0`** throughout (CSS var `--accent`, `--accent-glow`)
- Font: Inter (Google Fonts) — clean, modern, similar to Claude's aesthetic
- Gem-style tiles: radial gradient + highlight sheen + shadow
- **Raised tile selection**: No path line through tiles. Selected tiles lift up (4px vertical offset + 1.08x scale), get brighter gradient, deeper shadow, outer glow halo, and white border. Creates a tactile "raised" effect.
- **New High Score banner**: Green (#5BF080) pulsing text between header and info bar, shows when current score > saved best
- **No More Moves banner**: Red (#FF6B6B) persistent text at bottom, 20px bold, with hint text, fade-in animation
- **Final score**: White #E8E8F0 with letter-spacing 3px and text-shadow for pop
- **Easter eggs**: rare tile sparkle twinkle, soft color bloom (7+ chains), confetti on new high score, score block shake on epic chains, breathing honeycomb backdrop, chain record toast
- Chain bonus popups linger longer (1.2s–2.5s based on chain length)

## Key Design Decisions
- Mobile-first (375px viewport), safe area insets, no hover states
- Canvas for game board (performance), DOM for UI overlays (accessibility)
- No symbols on tiles — color only (removed for cleaner look)
- Trophy emoji button for leaderboard access
- Reshuffle button (↻) with count badge in info bar, pulses when stuck
- "Adult" aesthetic: muted jewel tones, no candy colors, sophisticated font
- Two modes only: Classic and Sprint (Daily mode removed)
- Dynamic HEX_SIZE calculation to fit any viewport (min 16px, max 28px)
- Chrome iOS zoom fixes: viewport meta, text-size-adjust, touch-action manipulation

## Key Constants
```javascript
const HEX_GRID_RADIUS = 3;    // 37 cells
const MIN_CHAIN = 3;
const NUM_COLORS = 6;
const FREEZE_AGE = 8;          // moves before tile freezes
const MAX_RESHUFFLES = 2;
const SPRINT_TIME = 45;        // seconds
const TIME_BONUSES = { 3: 0, 4: 2, 5: 4, 6: 7, 7: 10 };
const SCORE_TABLE = { 3: 150, 4: 400, 5: 850, 6: 1500, 7: 2500, 8: 4000, 9: 6500, 10: 10000 };
```

## Hosting
- GitHub repo: `ThinkingAndTinkering/hexchain`
- Render static site hosting
- Local testing: Python HTTP server on port 8080 + Cloudflare tunnel (`cloudflared tunnel --url http://localhost:8080`)
