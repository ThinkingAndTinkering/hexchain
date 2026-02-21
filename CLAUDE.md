# Hex Chain

## Project Overview
Browser-based mobile puzzle game. Single HTML file, no frameworks, no build step. Canvas 2D rendering for the hex grid, DOM for chrome/overlays.

## Architecture
- **Rendering**: Canvas 2D with `requestAnimationFrame` + dirty-flag system
- **Hex Grid**: Axial coordinates (q, r), pointy-top hexagons, grid radius 3 (37 cells)
- **Coordinate Math**: `axialToPixel` / `pixelToAxial` / `axialRound` for conversions
- **Gravity**: Hex cascade — cell-by-cell bottom-up pull. Each empty cell pulls from upper-left `(q, r-1)` or upper-right `(q+1, r-1)`, random tiebreak when both have tiles. Iterates until stable (max 10 passes). Diagonal settle animations with both X and Y offsets (550ms duration, 120ms stagger per pass — intentionally slow/visible).
- **Input**: Pointer Events (pointerdown/move/up) for cross-device touch-drag path drawing
- **Animations**: Time-based system with Maps: `clearAnims`, `settleAnims`, `appearAnims`, `highlightAnims` (glow before clear), `trailGlows`, `ripples`
- **Particles**: Sparkle effects on chain clears + ambient floating particles + confetti on new high scores
- **DPR-aware**: Canvas scales for retina displays

## Game Mechanics
- Draw paths through adjacent same-colored hex tiles (min chain: 3)
- Tiles clear along the path with staggered animation, gravity pulls tiles down diagonally (plinko-style), new tiles fill gaps
- **Freeze mechanic** (Classic only): Each tile has an `age` counter. Base freeze age is 7 moves. Progressive difficulty: freeze age drops to 6 at move 15, to 5 at move 30, to 4 at move 50. Countdown numbers (3, 2, 1) show on tiles approaching freeze. Pulsing colored border on tiles 1 move from freezing. Toast notification at each difficulty increase warns "Tiles are aging faster!"
- **Chain reward tiers (v3) — sequential multi-phase animation**:
  - **4+ chain → Color Wipe**: Chain clears first (gravity + refill). THEN after 500ms pause, remaining tiles of that color get highlighted (700ms subtle glow), then slowly fade/shrink away, then gravity + refill fills those gaps. Bonus points (50 per wiped tile), float text "COLOR WIPE!" (matches nova style). Frozen tiles of that color ARE included in wipe. Wildcard tiles are NEVER wiped. Nova tiles of matching color ARE wiped.
  - **6+ chain → Color Wipe + Same-Color Reset** (Classic only): All tiles of that color get age reset to 0
  - **7+ chain → Color Wipe + Global Reset** (Classic only): ALL tiles of ALL colors get age reset to 0, "BOARD RESET!" float text

### Special Tiles
- **Wildcard tile** (`TILE_WILD`): 6-colored hex with each wedge showing one of the 6 game colors + white center highlight + ✦ icon. Matches ANY color in chains. Spawns every 8-10 moves (randomized) as a refill tile. Path color is adopted from the first non-wild tile in the chain. **Never freezes. Never removed by color wipe** (only cleared if part of a chain).
- **Nova tile** (`TILE_NOVA`): **DISABLED** via `NOVA_ENABLED = false` feature flag. All code retained — set flag to `true` to re-enable. Design: horizontal white stripes + pulsating white border. When enabled: earned after 5+ chain, clears 6 adjacent tiles on chain, never freezes, included in color wipe.
- **Lightning tile**: REMOVED (was too confusing with wildcard + nova)

### Other Mechanics
- **Streak multiplier**: Consecutive 5+ chains build a score multiplier: 1x → 2x → 3x → 4x... (whole number steps). Resets on a sub-5 chain or reshuffle. "Nx STREAK!" float text shown. Multiplier displayed in chain info bar.
- **Manual reshuffle**: 1 reshuffle per game (reduced from 2). Button in info bar with ↻ icon + count. When no valid chains exist, reshuffle button pulses and persistent red "NO MORE MOVES" banner shows. Resets streak.
- **Classic mode**: Unlimited moves, 6 colors. Frozen tiles cause board lockup. No timer. Shows Chains counter.
- **Sprint mode**: 45-second timer, 6 colors, NO freezing. Timer starts on first chain. Long chains earn bonus time: 4-chain +1s, 5-chain +2s, 6-chain +3s, 7+ chain +5s.
- Chain tier feedback: Nice! (3) → Great! (4) → Amazing! (5) → Fantastic! (6) → INCREDIBLE! (7) → LEGENDARY! (8) → GODLIKE!! (10+)
- Deadlock detection via flood-fill connected components (excludes frozen tiles, wildcards bridge adjacent colors)
- **Game Over phrases**: Random selection from 50 hex-themed puns/phrases (e.g., "Hexcellent!", "HEX YEAH!", "Peak hexformance!", "Next-level hextelligence!")

## Tile Colors (8 defined, 6 used in all modes)
Slate Blue (#5B8CB8), Sage (#5A9E7E), Ochre (#C8A855), Terracotta (#B85C5C), Plum (#8B6BAA), Copper (#C47850), Fern (#6AAA5A), Pewter (#A0A0A8). All have base/dark/light variants for gem gradient rendering.

## Scoring
3-chain: 150, 4: 400, 5: 850, 6: 1500, 7: 2500, 8: 4000, 9: 6500, 10: 10000, 10+: exponential (~×1.6 per additional tile). Streak multiplier applied on top. Color wipe bonus: 50 per wiped tile. Nova bonus: 30 per adjacent tile cleared.
Target ranges: fast game 5-10K, good game 10-30K, great game high 5 digits.

## Analytics (PostHog)
- **Provider**: PostHog (US region, anonymous person profiles)
- **API key**: `phc_5VCEQ9CpEHv0cWi30504ICLtYLp1ZHYjIGNz2PMjDdb`
- **Host**: `https://us.i.posthog.com`
- **Autocapture**: enabled (clicks, pageviews, page leaves)
- **Custom events tracked**:
  - `game_start` — mode, returning_player
  - `game_over` — score, best_chain, total_cleared, total_chains, moves, duration_seconds, reshuffles_used, is_new_high_score
  - `chain_completed` — length, points, color, mode, triggered_color_wipe, triggered_color_reset, triggered_global_reset, move_number, streak_count, streak_multiplier
  - `reshuffle_used` — reshuffles_remaining, mode, score_at_reshuffle, move_number
  - `tutorial_dismissed` — first_time
- **Helper**: `track(event, props)` wraps `posthog.capture()` with try/catch

## Storage
- localStorage for high scores per mode, tutorial seen flag, leaderboard (top 100 entries with timestamps, filtered by mode Classic/Sprint + period today/month/all-time for display), sound preference (`hexchain-sound`), music preference (`hexchain-music`)
- **Leaderboard highlight**: Most recent game's score is highlighted with accent color glow and "Now" date label if it made the current filtered list

## Audio & Haptics
- **Audio engine**: Web Audio API for SFX (fully synthesized, no audio files). HTML5 Audio for background music (`background-music.mp3`).
- **Default OFF** — no AudioContext created until user taps sound button. Setting persisted in localStorage.
- **Sound toggle**: 🔊/🔇 button in header bar (below title). Lazy-inits AudioContext on first user gesture (browser autoplay policy).
- **SFX** (12 effects): tile select (ascending blips), path reject (low buzz), chain clear (pentatonic ascending per tile), score pop (bright pluck), streak activation (rising filtered sweep), new high score (major chord arpeggio), tile freeze (crack + noise), difficulty increase (ominous minor 2nd), game over (descending phrase), color wipe (noise whoosh), reshuffle (random pitch flurry), timer tick (urgent clicks when ≤10s)
- **Background music**: `background-music.mp3` — loops via HTML5 Audio element, volume 0.3. Replace the file to change the music.
- **Haptic vibration**: `navigator.vibrate()` wrapper — Android only, silently ignored on iOS. Short pulses on chain clears, buzz on game over, patterns for color wipe/high score/freeze/streak.
- **Node graph**: Oscillators → per-voice GainNode → sfxGain (0.7) → masterGain (1.0) → destination.

## Visual Design
- Premium dark theme: `--bg-dark: #141419`, radial gradient background
- **Steel blue accent `#7EB8D0`** throughout (CSS var `--accent`, `--accent-glow`)
- Font: Inter (Google Fonts) — clean, modern, similar to Claude's aesthetic
- Gem-style tiles: radial gradient + highlight sheen + shadow
- **Raised tile selection**: No path line. Selected tiles lift up (4px offset + 1.08x scale), brighter gradient, deeper shadow, outer glow halo, white border.
- **Highlight animation**: Tiles targeted by color wipe or nova get a subtle white glow overlay (alpha 0.2) that pulses over 700ms before they clear. Uses `highlightAnims` map. Clearing is scheduled separately via setTimeout for sequential flow (no onComplete).
- **New High Score banner**: Green (#5BF080) pulsing text between header and info bar
- **No More Moves banner**: Red (#FF6B6B) persistent text, 20px bold, fade-in animation
- **Final score**: White #E8E8F0 with letter-spacing 3px and text-shadow
- **Special tile visuals**: Wildcard = 6-color wedge hex + ✦ icon. Nova = bold horizontal white stripes (85% opacity) alternating with tile color + prominent pulsating white border (no icon). Both visually distinct from regular tiles.
- **Empty cell fill**: Empty grid cells get a white fill (8% opacity) + white border (14% opacity) so gaps from color wipe/nova are clearly visible as ghost hexes. Skips cells with active clear or appear animations.
- **Easter eggs**: rare tile sparkle twinkle, soft color bloom (7+ chains), confetti on new high score, score block shake on epic chains, breathing honeycomb backdrop, chain record toast
- **Tutorial**: Scrollable on small viewports (overflow-y auto, max-height 100dvh)

## Key Design Decisions
- Mobile-first (375px viewport), safe area insets, no hover states
- Canvas for game board (performance), DOM for UI overlays (accessibility)
- No symbols on regular tiles — color only (special tiles get icons)
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
const FREEZE_AGE = 7;          // base moves before tile freezes (6 after move 15)
const MAX_RESHUFFLES = 1;
const SPRINT_TIME = 45;        // seconds
const NOVA_ENABLED = false;    // feature flag for nova tiles (disabled)
const WILDCARD_INTERVAL = 8;   // wildcard every 8-10 moves
const TIME_BONUSES = { 3: 0, 4: 1, 5: 2, 6: 3, 7: 5 };
const SCORE_TABLE = { 3: 150, 4: 400, 5: 850, 6: 1500, 7: 2500, 8: 4000, 9: 6500, 10: 10000 };
```

## Version History
- **v1 chain tiers**: 5+ color wipe, 6+ global thaw (age → FREEZE_AGE-3), normal chains thaw+refresh same-color
- **v2 chain tiers**: 4+ color wipe, 5+ same-color reset (age → 0), 6+ global reset (age → 0)
- **v3 chain tiers (current)**: 4+ color wipe (highlight-then-clear), 6+ same-color reset, 7+ global reset
- **Special tiles added**: Wildcard (6-color hex, every 8-10 moves), Nova (gold, earned after 5+ chain). Lightning removed.
- **Balance changes**: FREEZE_AGE 8→7, reshuffles 2→1, progressive difficulty at moves 15/30/50, streak multiplier for 5+ chains, Sprint time bonuses halved
- **Color wipe rework**: Changed from simultaneous to sequential multi-phase (chain clear → gravity → pause → highlight → slow fade → gravity). Reduced highlight brightness. Frozen tiles included in wipe. Special tiles (wildcard/nova) protected from wipe and freezing.
- **Nova + color wipe sequential flow**: Nova highlight (700ms) → disappear → gravity → color wipe highlight (700ms) → disappear → gravity. Each phase fully completes before the next begins.
- **Nova visual redesign**: Removed yellow pulsating sun icon. Now uses bold diagonal white stripes (55% opacity) clipped to hex + prominent pulsating white border (2.5px).
- **Gravity animation slowed**: Settle duration 320ms→550ms, cascade stagger 70ms→120ms per pass. More visible plinko-style movement.
- **Color wipe notification**: Uses float text ("COLOR WIPE!") matching nova style. Both effects use same notification approach.
- **Audio + haptics added**: 12 synthesized SFX (Web Audio API), procedural ambient music (3 oscillators + LFO filter sweep), haptic vibration (Android). Sound toggle in header, default OFF, persisted in localStorage.

## Hosting
- GitHub repo: `ThinkingAndTinkering/hexchain`
- Render static site hosting
- Local testing: Python HTTP server on port 8080 + Cloudflare tunnel (`cloudflared tunnel --url http://localhost:8080`)
