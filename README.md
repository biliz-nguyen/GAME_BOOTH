# FLOÉ Joy Arcade

FLOÉ booth arcade: 20-card **Flip the Charm** memory game and interactive 3D **Catch Your Joy** claw machine.

## Rules (October 2026)
- All 20 cards are shown for 6 seconds.
- The memory phase lasts **20 seconds**, with no restriction on the number of card-pair attempts.
- **5+ matching pairs** unlock the 3D claw machine.
- **3–4 matching pairs** show a result of **2 candies**.
- **0–2 matching pairs** show a consolation message.
- Players can **start another round any time**: no per-browser or per-person lock.
- The claw machine allows **another claw attempt if the previous attempt missed** (caught=false). A successful catch ends that claw sequence, displays a prize, and the player may start a fresh round.
- On a successful catch, the browser randomly selects a prize with fixed weights: **100 voucher 5%, 40 regular keychains, 1 handmade keychain** (total 141). These are weights, not live inventory.

## Site and deployment
- Repository: https://github.com/biliz-nguyen/GAME_BOOTH
- Vercel project: `floe-game-booth`
- Production domain: https://floe-game-booth.vercel.app
- Default Git branch: `main`.
- Precompiled static website (`index.html`, `assets/`). No build step needed.
- Includes FLOÉ logo assets, Vietnamese font assets, cards, 3D scene and mobile controls.

## Operational notes
- **No entry codes, OTP, QR tokens, personal identifiers, or participation restrictions.** A shared QR can point directly to the production homepage.
- **No automatic prize inventory deduction or prize code issuance.** Prizes are announced on the phone screen and managed manually by booth staff.
- **No external Supabase call is made by the production game.** Legacy Supabase structures from earlier prototypes are not part of this public game flow.
- Because results and prize randomization are handled in client JavaScript, screenshots can be reused and results can be manipulated. Staff must independently decide how physical gifts are issued. Unlimited replays do **not** enforce prize-claim limits.
- The numerical prize weights (100/40/1) do not decrease if physical items run out. Change the prize logic manually as needed.

## Notes for maintainers
- The existing compiled JS bundle is `assets/index-Bb9PN9JV.js`. Keep HTML asset references consistent after edits.
- Before deploying any edits, check JS syntax: `node --check assets/index-Bb9PN9JV.js`.
- Test memory timing, success and failure results, 3D/fallback rendering, repeated claw misses, successful catches, new round, and mobile layout.
