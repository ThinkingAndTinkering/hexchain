# Hex Chain

## Project Overview
Browser-based mobile puzzle game. Single HTML file, no frameworks, no build step. Canvas 2D rendering for the hex grid, DOM for chrome/overlays.

## Architecture
- **Rendering**: Canvas 2D with `requestAnimationFrame` + dirty-flag system
- **Hex Grid**: Axial coordinates (q, r), pointy-top hexagons, grid radius 3 (37 cells)
- **Coordinate Math**: `axialToPixel` / `pixelToAxial` / `axialRound` for conversions
- **Gravity**: Columns defined by `2*q + r = constant` (visual vertical alignment)
- **Input**: Pointer Events (pointerdown/move/up) for cross-device touch-drag path drawing
- **Animations**: Time-based system with Maps tracking clearing/settling/appearing/ripple states
- **Particles**: Sparkle effects on chain clears + ambient floating particles + gold confetti on new high scores
- **DPR-aware**: Canvas scales for retina displays

## Game Mechanics
- Draw paths through adjacent same-colored hex tiles (min chain: 3)
- Tiles clear along the path with staggered animation, gravity pulls tiles down, new tiles fill gaps
- **Freeze mechanic**: Each tile has an `age` counter. After 7 moves without being cleared, tiles freeze (icy overlay + snowflake, can't be used in chains). Countdown numbers (3, 2, 1) show on tiles approaching freeze. Pulsing border on tiles 1 move from freezing.
- **Unfreeze mechanic**: When a chain clears adjacent to a same-color frozen tile, the frozen tile also gets cleared (no points awarded). Toast notification says "Thawed a frozen tile!"
- **Manual reshuffle**: Players get 2 reshuffles per game (button in info bar with ↻ icon + count). When no valid chains exist, a toast prompts the player to use reshuffle. If no reshuffles remain, game ends.
- **Classic mode**: Unlimited moves, starts with 5 colors, adds a color every 6 chains (up to 8). Frozen tiles + color escalation cause board lockup.
- **Sprint mode**: 45-second timer, 6 colors. Timer starts on first chain. Long chains earn bonus time: 4-chain +2s, 5-chain +4s, 6-chain +7s, 7+ chain +10s.
- **Daily mode**: Sprint with seeded PRNG (mulberry32) based on date
- Chain tier feedback: Nice! (3) → Great! (4) → Amazing! (5) → Fantastic! (6) → INCREDIBLE! (7) → LEGENDARY! (8) → GODLIKE!! (10+)
- Deadlock detection via flood-fill connected components (excludes frozen tiles)

## Tile Colors (8 total, muted earth/jewel tones)
Slate Blue (#5B8CB8), Sage (#5A9E7E), Ochre (#C8A855), Terracotta (#B85C5C), Plum (#8B6BAA), Copper (#C47850), Fern (#6AAA5A), Pewter (#A0A0A8). Starting 5 in Classic, all 6 in Sprint. All have base/dark/light variants for gem gradient rendering.

## Scoring
3-chain: 300, 4: 800, 5: 1700, 6: 3000, 7: 5000, 8: 8000, 9: 13000, 10: 20000, 10+: exponential (~×2 bump from previous values)

## Storage
- localStorage for high scores per mode, tutorial seen flag, leaderboard (top 100 entries with timestamps, filtered by today/month/all-time for display)

## Visual Design
- Premium dark theme: `--bg-dark: #141419`, radial gradient background
- Softened gold accent `#D4AA60` throughout
- Font: Inter (Google Fonts) — clean, modern, similar to Claude's aesthetic
- Gem-style tiles: radial gradient + highlight sheen + shadow
- Touch-drag path: triple-layer glow line (outer glow → mid color → white core)
- Screen shake + extra particles for epic/legendary chains
- **Easter eggs**: rare tile sparkle twinkle, trail afterglow (5+ chains), ripple wave (7+ chains), gold confetti on new high score, score block shake on epic chains, breathing honeycomb backdrop, chain record toast
- Chain bonus popups linger longer (1.2s–2.5s based on chain length)

## Key Design Decisions
- Mobile-first (375px viewport), safe area insets, no hover states
- Canvas for game board (performance), DOM for UI overlays (accessibility)
- No symbols on tiles — color only (removed for cleaner look)
- Trophy emoji button for leaderboard access
- Reshuffle button (↻) with count badge in info bar
- "Adult" aesthetic: muted jewel tones, no candy colors, sophisticated font

## Serving for Mobile Testing
- Python HTTP server on port 8080 + Cloudflare tunnel (`cloudflared tunnel --url http://localhost:8080`)
