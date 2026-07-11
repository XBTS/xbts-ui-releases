## [2.9.0] - 2026-07-08 - Price providers (Binance) + AUTO start-price toggle

### Added
- **External price providers:** the Grid Bot's reference price can now come from an external exchange (Binance) instead of the on-chain Bitshares ticker. New `priceProviders.js` (pure fetch + symbol mapping) and `priceProviderStore.ts` (global default provider, per-pair overrides, manual exchange-symbol overrides, API-key storage). Binance prices are pulled from the CORS-friendly public domain `data-api.binance.vision` (no API key required for price data).
- **Bot settings modal (`BotSettingsModal.vue`):** opened via a new gear icon in the Control Console header. Lets the user pick the global price source, override the source or exchange symbol per pair, test the connection live, and store optional exchange API credentials (kept locally for the user's own build).
- **AUTO start-price toggle:** the START PRICE field's AUTO switch is fully wired again - when ON (and the bot is idle) the live market price is auto-mirrored into START PRICE (pair is never changed); turning it OFF frees the field for manual entry. Added the missing `bot.autoprice*` i18n keys.

### Changed
- `fetchCurrentPrice()` (botEngine.js) is now provider-aware - the single choke point routes BOTH bot setup and the live runtime through the selected provider, with transparent fallback to the on-chain Bitshares price when a pair isn't listed on the exchange (e.g. STH → 400 → on-chain).

### Fixed
- Restored the AUTO price behaviour the previous refactor had detached (`autosync` was referenced in the template but no longer bound in the setup scope, and the live ticker never fed START PRICE).

## [2.8.4] - 2026-07-08 - Unlock modal: auto-unlock + no password autosave

### Added
- **Auto-unlock:** the PIN unlock modal now unlocks automatically the moment the correct PIN is entered - no need to press the Unlock button. (Validation is a cheap hash compare, checked per keystroke.)

### Changed
- **No browser "save password" / autofill on the PIN field:** the input is now `type="text"` (so browsers/password managers don't treat it as a credential) with the value visually masked via CSS `-webkit-text-security: disc`. Also added `data-lpignore` / `data-1p-ignore` / `autocomplete="off"` to suppress managers.

### Fixed
- TODO

## [2.8.3] - 2026-07-08 - Grid Bot: softer order-book colors (muted green / rose)

### Changed
- **Softer /bot palette:** replaced the harsh neon buy/sell colors (`#00ff66` / `#ff0055`) with a muted green (`#4fb87a`) and soft rose (`#d96a86`) across the whole Grid Bot page - ladder prices/bars, chips, badges, buttons and the balance-split - to match the main DEX order-book styling. All hardcoded neon `rgba()` tints/glows were remapped to the soft palette too.

### Added
- TODO

### Fixed
- TODO

## [2.8.2] - 2026-07-08 - Grid Bot: ladder volumes use base precision; log Errors filter excludes warnings

### Changed
- **Ladder volume precision:** the order-book volume column (BASE units) now renders with the **base asset's own `precision`** decimals instead of a fixed cap, matching the price-precision change.

### Fixed
- **Execution-log "Errors" filter caught warnings:** entries like `ANTI-SPAM: rebalance deferred` and `MONITOR STOPPED` (log type `warn`) were shown under **Errors**. The Errors filter now matches only genuine failures (type `bad`); `rebalance deferred` now falls under **Rebalances** and status warnings appear only under **All**.

### Added
- TODO

## [2.8.1] - 2026-07-08 - Grid Bot: sync-pair reads real active market; ladder prices use quote precision

### Changed
- **Ladder price precision:** virtual order-book prices now render with the **quote asset's own `precision`** decimals instead of a fixed 4 digits, so low-value / high-precision assets are no longer clipped (e.g. `BTS Price 1.36524` instead of `1.3652`).

### Fixed
- **"SYNC PAIR" pulled a phantom pair:** it read `marketStore.selectedTradingPair`, which is legacy and never populated by the current UI, so it synced a stale value (e.g. `LTC/USDT`) and wiped the bot's actual pair. It now reads the live spot terminal's active market (`settingsStore.lastTradingPair`, a `BASE_QUOTE` string), with the legacy field kept only as a fallback.

## [2.8.0] - 2026-06-29 - Grid Bot: layout redesign, anti-fee regroup, stop-cancels-orders, refactors

### Added
- **Portfolio menu item** replacing "Explorer" in the top nav (Wallet icon): links to the signed-in user's `/account/<username>`. Hidden for guests (new `requiresAuth` flag on menu items). i18n `Portfolio` (en/ru).
- **Execution Log filters** in the /bot log tab: `All / Fills / Rebalances / Errors` chips with per-category counts, plus a total-event count badge on the LOG tab button.
- **Stop now cancels open orders:** stopping the bot builds a cancel-only plan for the **current pair only** (`buildCancelAllPlan`) and broadcasts it in one transaction, so leftover grid orders no longer distort the recalculated ladder. Best-effort (failures logged, never block the stop). STOP button shows a `STOPPING…` state.
- **Quote-asset price label** on every virtual order-book row (`<QUOTE> Price <value>`, e.g. `USDT Price 0.00134942`) so it's clear the left column is a price. i18n `price_word`.

### Changed
- **/bot layout redesign:** the market/pair selector moved to the **top** of the Control Console; the `GRID VISUALIZER` right panel is now **tabbed** - a **Strategies** tab (strategy selector on the left half, settings - Grid Step, Grid Levels, Distribution, description - on the right half) and an **Execution Log** tab. Grid Levels moved out of the left console into the Strategies tab.
- **EST PROFIT / CYCLE** moved into the `GRID VISUALIZER` header (green-bordered box, between the title and the CANCEL/BUY/SELL chips).
- **Control panel header** dropped the "CONTROL & STATUS" label, keeping only the WEB_WORKER status badge.
- **Softer strategy buttons:** the active Strategy/Distribution buttons no longer use the harsh solid neon-green fill - they now use a tinted-outline active style (green text + border + subtle tint).

### Fixed
- **Anti-fee regroup guard:** Grid & Staggered no longer auto-rebalance on price **drift alone**. Previously the bot cancelled + recreated the whole ladder every ~minute whenever price left the spread even with **zero fills**, burning blockchain fees. Auto-regroup now requires at least one (partial) fill; pure drift only flags a manual REGROUP. Relative (follows price) and KOTH (own peg loop) are unchanged.
- **WebSocket connect stall on first load:** `nodeStore._connectTo` had no timeout, so a dead/slow preferred node (e.g. a stale personal LAN node) could hang in CONNECTING forever, leaving the app stuck until a manual F5. Added an 8s connect-timeout with automatic node failover (and swallowed the late race rejection).

### Engineering
- Extracted from the ~1500-line `Bot.vue`: pure formatters → `services/dexbot/format.js`; the teleported pair selector → `components/bot/BotPairPicker.vue`; the virtual order-book (ladders + current-price divider + pair-balance split) → `components/bot/BotLadder.vue`. `Bot.vue` down to ~1200 lines. No behavior change (logic stays in `Bot.vue`; children are presentational).
- `vite.config.ts`: excluded `public/svg` + `public/fonts` from the dev file-watcher to stay under the container's `fs.inotify` watch limit (was crash-looping the dev server with ENOSPC).

## [2.7.0] - 2026-06-28 - Secure sessions, multi-account isolation, KOTH & Staggered live

### Added
- **King of the Hill (KOTH) - live execution:** WSS-driven pegging loop that places one tick better than the best competitor / holds when leading / sleeps under the minimum spread.
- **Staggered Orders - live execution:** graded volume cascade (Mountain/Neutral/Valley) with portfolio-weight auto-balancing that biases deployable funds toward the deficit asset on each fill/rebalance.
- **Login screen background** (`login-bg.png`).

### Changed
- **Session security:** private keys are decrypted **in-memory only** (never persisted back to storage); read-only hydration on unlock. AES key derivation upgraded from a plain SHA-384 hash to **PBKDF2**.

### Fixed
- **Multi-account bot isolation:** bot state no longer leaks between different signed-in accounts.
- Balance-ratio split-bar visual overlap in the Grid Visualizer.

## [2.6.5] - 2026-06-27 - /bot uses the site font (Outfit); drop unused JetBrains Mono

### Changed
- **/bot now uses the same font as the rest of the site** (`'Outfit', sans-serif` = `$body-font`) instead of the JetBrains Mono monospace stack - for the Grid Bot root, the numeric term inputs, and the teleported pair-picker modal.
- **Removed the now-unused self-hosted JetBrains Mono** (preload + `jetbrains.css` link in `index.html` and the `public/fonts/jetbrains/` woff2 files, ~180 KB) since /bot was its only consumer. One less asset loaded site-wide; still zero external font requests.

## [2.6.4] - 2026-06-27 - Pair balance ratio bar in Grid Visualizer

### Added
- **Pair balance ratio split bar** (`[data-testid=bot-balance-split]`) under the /bot Grid Visualizer ladder: a market-depth-style green/red bar showing what share of the pair's balance is held in the BASE asset vs the QUOTE asset, with both legs valued in the QUOTE currency (e.g. STH vs USDT, valued in XBTSX.USDT) at the live price (falls back to start price). Each segment shows its % + symbol (label hidden under 12% width) and the bar foot shows the per-leg quote value. Reuses the same portfolio-weight concept as the Staggered strategy - instantly shows which asset is over/under-weighted. Added i18n `pair_balance` / `valued_in` (en/ru).

## [2.6.3] - 2026-06-27 - Fix Bot crash on rebalance (i18n) + clearer over-balance UX

### Fixed
- **CRITICAL: /bot went black after an order filled / on reload.** The `bot.rebalanced` toast/status message contained a literal `@` (`"Rebalanced @ {price}…"`), which vue-i18n parses as a linked-message marker → `SyntaxError: Invalid linked format` thrown during render → the whole Bot component crashed (black page). Because `lastRun` is persisted, it re-crashed on every mount even after reload. Escaped the `@` via vue-i18n literal interpolation `{'@'}` in both en/ru. Verified with the @intlify message-compiler (old string throws, fixed string compiles). No storage reset needed - once redeployed the persisted state renders fine again.

### Changed
- **Clearer "exceeds balance" UX:** when a fund amount is larger than the available balance, the offending fund input now highlights with a pulsing red border and its label turns red, and the warning became a one-tap **"⚠ EXCEEDS BALANCE - tap to use max <bal>"** button that instantly sets the fund to 100% of the balance. The right-side BLOCKED message now points to "the highlighted field on the left" so it's obvious what to fix.

## [2.6.2] - 2026-06-27 - Explicit account loading indicator

### Changed
- **Account page first-load feedback:** the full-page `AccountSkeleton` now shows an explicit animated indeterminate progress bar + a spinner and the label "Идёт загрузка информации аккаунта…" / "Loading account information…" (`[data-testid=account-loading-indicator]`), on top of the existing shimmer rows. Previously the load window looked like a blank/empty table, making users think the app was stuck. Added `account.loading_info` i18n key (en/ru).

## [2.6.1] - 2026-06-27 - Universal persistence adapter (web/desktop) + Bot config to Sled

### Added
- **Universal JSON persistence layer** in `storageAdapter.ts`: `getJSONSync()` / `setJSON()` / `removeKey()` / `hydrateKey()`. One API for ANY new key/object with no per-type code and no platform branching at the call site - WEB writes localStorage (sync); DESKTOP keeps a localStorage mirror for instant sync reads **and** durably write-throughs to the Sled KV store (survives webview-localStorage resets). `hydrateKey()` gives snapshot-first load on desktop. The dedicated heavy-context caches (wallet/asset/node trees) remain; everything else now uses this generic layer.
- **Bot state is now desktop-durable:** `botStore` persists its config (incl. the new `strategy` / `minSpread` / `distributionMode` fields) and session history through the universal layer, and added `hydrateDesktopCache()` (called from /bot `onMounted`) for snapshot-first load from Sled on desktop.

### Fixed
- v2.6.0 regression: a TypeScript `as` cast leaked into the plain-JS `<script setup>` of `Bot.vue`, crashing SFC compilation and blanking the entire /bot page. Replaced with valid JS (`[...LIVE_STRATEGIES].includes(...)`).

## [2.6.0] - 2026-06-27 - Grid Bot strategy engine: switching + Relative Orders + Staggered skeleton

### Added
- **Strategy selector in the Control Console** (`[data-testid=bot-strategy-seg]`): `Default Grid | Relative Orders | Staggered | King of the Hill`. Left-panel inputs now show/hide dynamically per strategy.
- **Relative Orders (fully live):** Grid Levels locked to 2 (1 BUY + 1 SELL, input hidden). Re-centers on every fill using the price right after execution and re-places the pair; uses a short 5s anti-spam window (vs the grid's 60s) so it follows liquidity here-and-now. Reuses the existing fill→rebalance path.
- **Staggered Orders (Phase 3 skeleton, preview-only):** `DISTRIBUTION MODE: Mountain (1.0) | Neutral (0.5) | Valley (0)` selector. Volume cascade in `orderPlanner.applyStaggeredDistribution()` - each level's volume is re-weighted by `pow(i, exponent)` (i = 1-based distance from center) with total per-rail funds preserved (Mountain=linear, Neutral=sqrt, Valley=flat). Also added `computePortfolioWeights()` - the documented "smart balance spring" (asset-allocation rebalancing) skeleton. Execution deferred to Phase 3; Launch disabled (preview only).
- **King of the Hill (UI + pure decision core, live pegging deferred):** added `Minimum Allowed Spread` input + new `kothCore.computeKothPeg()` pure decision function (peg one tick better than the best competitor / hold when already leading / sleep when spread < min). Live order-book pegging in the worker ships next release; Launch disabled this release (preview only).

### Engineering
- `botStore`: `config.strategy` + `minSpread` + `distributionMode`; `botPlanParams()` helper (Relative forces 2 levels); exports `STRATEGIES` / `LIVE_STRATEGIES`. Only Grid + Relative are launchable this release.
- Pure cores stay environment-agnostic and were unit-validated via a standalone node run (14/14 assertions: KOTH peg decisions + staggered cascade monotonicity & fund preservation).

## [2.5.4] - 2026-06-27 - Resilient chunk-load recovery (auto-reload on dynamic import failure)

### Added
- **Auto-recovery from failed route/chunk loads:** after a redeploy (or a transient CDN/origin error such as Cloudflare 522), an old `index.html` can reference a `*.js` chunk that is missing/unreachable, causing `Failed to fetch dynamically imported module` and a broken page on navigation. Added an additive `router.onError` handler + a Vite `vite:preloadError` listener that perform a single, loop-guarded (`sessionStorage`) full reload to the target path so users no longer have to manually hard-refresh (Ctrl+R). Triggers only on a genuine chunk-load failure - working flows are untouched.

### Notes
- Root cause of the reported `/trading` (terminal) failure was a server-side Cloudflare 522 (origin timeout), not an app bug. This change makes the SPA self-heal from such transient/stale-deploy cases.

## [2.5.3] - 2026-06-27 - Fix /bot CSP error: self-host JetBrains Mono (no external deps)

### Fixed
- **/bot production console errors / broken CSS preload:** the Grid Bot scoped style had a remote `@import url('https://fonts.googleapis.com/...JetBrains+Mono')` which the CSP (`style-src 'self' 'unsafe-inline'`) blocked, which in turn made Vite fail with `Unable to preload CSS for /assets/Bot-*.css`. Removed the external import and **self-hosted JetBrains Mono** (latin + cyrillic, weights 400/500/700/800, ~180 KB woff2) under `public/fonts/jetbrains/` + `public/fonts/jetbrains.css`, linked from `index.html` (mirrors the existing self-hosted Outfit setup). The cyber terminal aesthetic is now preserved offline with zero external font requests.

### Notes
- The remaining `static.cloudflareinsights.com/beacon.min.js` CSP message is injected by the Cloudflare proxy at the hosting layer (not by the app code) and is harmless - it is blocked by CSP and does not affect the app.
- Policy going forward: bundle everything locally (fonts, libs) - no external/CDN runtime dependencies. FontAwesome already loads locally via npm; the old CDN `<link>` in index.html stays commented out.

## [2.5.2] - 2026-06-27 - Grid Bot light-theme support

### Added
- **/bot light theme:** the Grid Bot page now reacts to the global dark/light theme switcher (`html[data-theme='light']`). Added a light palette override (re-maps `--bg/--panel/--line/--buy/--sell/--muted/--txt/--warn` plus the few hardcoded dark surfaces: matrix gradient, ladder mid-divider, depth/bar backgrounds, steel button) and a light variant for the teleported pair-picker modal. Cyberpunk dark remains the default; trading semantics (green BUY / red SELL) preserved with darker, white-readable shades in light mode.

## [2.5.1] - 2026-06-27 - Desktop-aware Grid Bot note + full Reset-all wipe (SLED/localStorage)

### Added
- **Full "Reset all" wipe:** Settings → Reset all now does a complete storage wipe. New `storageAdapter.wipeAll()` → DESKTOP clears the entire Sled DB via new Rust IPC command `wipe_cache` (clears all 5 trees + flush); WEB does `localStorage.clear()`. Wired into `ResetSection.handleReset` (after `accountStore.resetAll()`).

### Changed
- **/bot:** the "// WEB RUNTIME LIMITATIONS" panel (tab-sleep / session-limit warnings + Download Desktop CTA) is now hidden on the desktop (Tauri) build - it only applies to the web runtime. Gated by `isDesktop()` (`v-if="!isDesktopApp"`).

## [2.5.0] - 2026-06-27 - Phase 2 desktop SLED snapshot-first cache + footer LIVE/CACHED badge

### Added
- **Tauri + SLED snapshot-first cache (desktop):** isomorphic `storageAdapter.ts` (`isDesktop()` → SLED IPC on desktop, `localStorage`/no-op on web). Rust side (`src-tauri/src/db.rs` + `lib.rs`) exposes IPC commands over 5 Sled trees (account/balances/assets_metadata/nodes/kv): `cache_wallet_state`, `get_cached_wallet_state`, `update_asset_meta`, `get_cached_assets`, `cache_account`, `get_cached_account`, `update_node_stat`, `get_cached_nodes`, `kv_get/set/remove`.
- **Snapshot-first hydration hooks (desktop-only, no-op on web):** `walletStore` persists balances + referenced asset meta and hydrates on launch; `nodeStore` persists node ping/stats and seeds pings from cache (`hydratedFromCache` flag); `assetStore` hydrates the token dictionary. Called from `App.vue onMounted` before `nodeStore.init()`.
- **Footer data-source badge** (`[data-testid=footer-data-badge]`): pulsing green **LIVE** when WSS is connected; amber **⚡ CACHED (SLED)** on desktop while serving the instant cache snapshot before WSS connects (badge never shows CACHED on web - no SLED).

### Changed
- `vite.config.ts`: added `server.watch.ignored: ['**/src-tauri/target/**']` so the Rust/Cargo build dir no longer overwhelms the Vite dev file watcher.
- Footer: removed the centered `© 2018 - <year>` copyright line.

### Notes
- Web build verified non-regressed (testing_agent iteration_29 = 100%): `/trading`, `/bot`, `/defi` render with live WSS block stream, zero fatal JS errors, all SLED hooks no-op on web.
- Rust `cargo check` could not complete in this container (missing system libs: pkg-config/webkit2gtk); desktop compile + live SLED hydration are verified locally by the user.

## [2.4.0] - 2026-06-26 - BitShares Browser Extension login + signing (Phases 1-4)

### Added
- **Browser Extension auth** (window.bitsharesWallet): "Browser Extension" button added in the Authentication Required modal next to "Import account" (single auth entry point). `walletStore.connectExtension()` connects, soft-checks chain id, fetches account+balances via WSS (NO private keys, NO PIN stored), sets an `sessionType: 'extension'` session.
- **Extension transaction signing:** `transactionStore.broadcastTransaction` routes to `broadcastViaExtension` for extension sessions → the extension popup signs AND broadcasts (`signTransaction({operations})`). All existing prepare/op-builders reused unchanged.
- **AWAITING_EXTENSION_SIGNATURE overlay** (`ExtensionSignOverlay.vue`, mounted in App.vue) shown while awaiting the extension popup; user rejection ({success:false}) is handled softly (no crash, gentle "cancelled" toast).
- Extension events wired: `accountChanged` (refresh balances), `locked`/`disconnect` (drop to guest).

### Changed
- `accountStore.setCurrentAccountName(name, recordLast=true)`: extension sessions pass `recordLast=false` so `lastAccountName` keeps the PIN account - PIN re-login no longer restores the extension account (bug avoided). `EnterPin` resets `sessionType='pin'` before key decryption. Keyless extension accounts skip `decryptCurrentAccountKeys`.

### Notes
- Full connect→sign flow requires the actual extension installed (load unpacked); not e2e-testable in this environment. UI integration, not-found handling and graceful errors verified; PIN/guest paths structurally unchanged.
- Footer now shows a "Connected via Extension: <account>" badge with a quick Disconnect button (visible only for extension sessions).



### Changed
- **DeFi wide layout scroll:** removed all sticky from the pools table (header now scrolls with the table - no more floating-header / under-nav artifacts). The left Fast Swap panel is rigidly pinned (`position: sticky` + its own `max-height/overflow`) so it no longer drifts with the right column.

### Added
- **Depth / Price toggle** in the pool mini-block: "Price" shows a lightweight SVG sparkline of the pair's recent price (1h buckets from market history) with green/red trend color and % change; "Depth" keeps the reserves split bar. APR moved into the stats row.



### Added
- **Pool depth mini-chart** under the Fast Swap panel (wide layout): clicking a pool row now shows the selected pair's reserves split bar (teal/indigo), APR, Liquidity (USD), 24h Volume (USD) and pool fee - visible before entering an amount.

### Fixed
- **DeFi sticky controls/header bug:** the pools column now scrolls inside its own container (`max-height: calc(100vh - 196px); overflow:auto`); search+chips stick at top:0 and the table header at top:92px. Set `.table-responsive { overflow: visible }` so the sticky `<thead>` resolves to the right scroll ancestor. This fixes the header floating mid-screen and the pinned controls sliding under the top nav on scroll.



### Added
- **Mobile terminal Chart tab:** removed the top Chart toggle (order book now sits higher); added a 4th "Chart" tab in the bottom sheet that expands to full and renders the price chart.
- **Bottom sheet 3 snap points:** collapsed / half / full. Tap handle cycles; drag snaps to nearest. Buy/Sell button stays visible in collapsed state.
- **DeFi wide layout (≥1380px):** Fast Swap shown inline on the LEFT (~35%, sticky) with pools table on the RIGHT; search + chips + table header pinned (sticky); clicking a pool row fills the swap pair. Extracted swap logic into `useDefiSwap` composable shared by the modal (narrow) and the new `DefiSwapPanel` (wide).
- **Account page skeleton on chunk load:** /account route uses `defineAsyncComponent({ loadingComponent: AccountSkeleton })` so no black screen while the lazy chunk loads.

### Changed
- DeFi "Fast Swap" button recolored to a dark theme-consistent style (was metallic/light).

### Fixed
- **WSS console error silenced:** `loadOpenOrders` retries up to 4× waiting for the socket and downgrades the transient `Still in CONNECTING state` from error → warn; open orders load reliably.



### Added
- **Bottom slide-up sheet (TradeLayoutMobile.vue):** the Open Orders / Savings / Deposit block is now a draggable bottom sheet with a grabber handle, drag gesture (touch up/down to expand/collapse), spring animation (cubic-bezier overshoot) and haptic feedback (navigator.vibrate) on snap. Tapping a tab auto-expands the sheet.

### Changed
- Sheet is collapsed by default to a ~92px peek anchored above the bottom nav, with `padding-bottom:150px` on the layout - the Buy/Sell button and Avail row are no longer overlapped (previous overlap bug fixed).
- **Open Orders cards (OpenOrders.mobile.vue)** restyled from flat gray to the cyber-navy theme (translucent dark bg, solid border, teal hover, green/red side labels).

### Fixed
- **WSS race in walletStore.loadOpenOrders:** it now waits for `connectionStatus==='connected'` (poll up to ~5s) before `getFullAccounts` and retries once on a transient socket error - fixes the recurring `WebSocket Still in CONNECTING state` that left open orders empty right after login.



### Fixed
- **Re-login by PIN showed guest mode:** after logout (which clears `currentAccountName`), unlocking the wallet with the PIN did not restore the active account, so /trading showed the guest/unauthorized state. Added persisted `accountStore.lastAccountName` (written on every login, kept through logout); `EnterPin.enterPin()` now restores `currentAccountName || lastAccountName` (re-arms walletStore + accountStore) and routes to /account/<name>. `Account.vue` onMounted also re-asserts `currentAccountName` for direct deep-links. Verified end-to-end (iteration_12.json).



### Changed
- **Mobile order form (OrderForm.mobile.vue)** fully redesigned to Binance style: dark rounded fields with inner label + centered value and subtle inline -/+ steppers; removed the old gray +/- boxes and gray percent buttons; replaced percent buttons with a diamond step slider (0/25/50/75/100, yellow track fill); cleaner Avail row + single Buy/Sell button.
- **Mobile top nav (Header.vue)** changed from cluttered colored icons to clean Binance-style text links (Trading / DeFi / Credits) with active underline; fixed left-clipping via `.page-main-header { justify-content: space-between }` on mobile.

### Fixed
- **Logout session not cleared:** walletStore.logout() now nulls currentAccountName (keeps encrypted accounts for re-login) and Profile.vue clears accountStore.currentAccountName → after logout /trading correctly shows guest/read-only mode instead of the logged-in account.
- **Account.vue blank gap:** added a proper initial-load skeleton (initialLoadDone + balance/refetch watchers + 6s fallback) so there is no empty area under the tabs right after login.



### Added
- Mobile top-left header quick navigation: TRADING / DEFI / CREDIT OFFERS links (Header.vue, mobile-only).
- "Connect Extension" placeholder button in the Authentication Required modal (GuestAuthModal.vue) - shows a "coming soon" toast (browser-extension integration deferred).
- Binance-style mobile trading bottom tabs: replaced "Market Trades" with **Savings** (pair assets + BTS balances) and **Deposit** (routes to /wallet/deposit/<asset>). (TradeLayoutMobile.vue)

### Changed
- Mobile bottom nav (Footer.vue): swapped "Trading" item for **SmartHolder** (/smartholder).
- Mobile toast containers pushed up 64px so they no longer overlap the bottom nav (_header.scss).

### Fixed
- Disabled `vite-plugin-vue-devtools` (gated behind ENABLE_VUE_DEVTOOLS=true): its floating button was overlapping and intercepting clicks on the SmartHolder bottom-nav item on mobile.


## [1.7.0] - 2026-06-25

### Added
- **Order confirmation now shows an estimated fill on desktop** (`PlaceOrderModal`): using the local incremental order book, it walks the book up to the limit price and shows the estimated average fill price, slippage % vs. the best price, and the immediately-fillable portion (or "Maker order - no immediate fill" for passive orders).

## [1.6.0] - 2026-06-25

### Added
- **Mobile order-review sheet**: tapping Buy/Sell on the mobile terminal now opens a bottom sheet that slides up with the full order summary (side, pair, price, amount, total, fee, expiry) and Confirm/Cancel - the order is only broadcast on Confirm. Desktop keeps the centered `PlaceOrderModal`.
- **Swipe Buy/Sell**: horizontal swipe left/right on the mobile order form switches between the Buy and Sell tabs, with a slide transition.
- **Light haptics** (`utils/haptics.ts`): a short vibration on Buy/Sell swipe and a success pulse when an order is placed (no-op on unsupported devices).
- **Cumulative order-book click on mobile**: clicking an order-book level now fills the form with the cumulative amount up to that level and that level's price, and auto-selects the matching Buy/Sell tab - matching desktop behaviour.

### Fixed
- Eliminated the Vue "onMounted/onUnmounted called with no active component instance" warnings by guarding `useDeviceType` and `useIdleTimer` with `getCurrentInstance()` (the idle timer is created from a watcher, i.e. outside setup); the idle timer now also cleans up its previous listeners.
- De-duplicated the market subscription: added a socket-generation token so `subscribe_to_market` runs once per pair per socket (re-subscribing only after a real reconnect); the log was also lowered to `debug`.

## [1.5.0] - 2026-06-25

### Added
- **Binance-style mobile trading terminal** (`TradeLayoutMobile.vue`, shown when viewport < 720px):
  - sticky pair header with last price and 24h change;
  - a lazy-loaded **Chart** toggle (the heavy chart mounts only when opened, keeping mobile fast);
  - the core **order book + Buy/Sell order form** row (price/amount/total with steppers, balance, 25/50/75/100% sizing);
  - bottom tabs **Open Orders / Market Trades** (recent market trades reuse the shared `TradeHistory` component);
  - reuses the incremental order-book engine and targeted subscriptions, with clean teardown on unmount.
  - Verified 8/8 (layout, chart toggle, tab switching, order-form math, percent buttons) with desktop regression intact.

## [1.4.0] - 2026-06-25

### Added
- **Incremental order-book diff engine.** The order book is no longer re-fetched on every market tick. `marketStore` now keeps an id-keyed map of live limit orders (seeded once via `getLimitOrders`) and applies deltas from the `subscribe_to_market` payload:
  - upsert on full `1.7.x` order objects, delete on removed-order id strings, and a throttled trade/ticker refresh on fill operations;
  - the displayed book (bids/asks + cumulative totals, up to 50 levels/side) is rebuilt locally with zero network per tick;
  - a 45s safety resync (full snapshot) corrects any drift;
  - seeds/re-seeds on market switch and after node reconnect.
  - Result: near-instant order-book updates with minimal traffic - verified 7/7 (deep monotonic book, correct totals, no crossed book, clean market/node switch).

## [1.3.0] - 2026-06-25

### Fixed
- **Revived the top ticker bar.** After the firehose removal the header price/volume indicators (BTC, BTS, HBD, …) went stale. Replaced the heavy 20s `fetchTopMarkets` poll (which re-fetched every known asset) with a focused live updater in `marketStore`:
  - a fast lightweight poll (~6s) that refreshes only the current top-N pairs (≈5 ticker calls);
  - a re-ranking pass every 10 minutes that re-evaluates which pairs are top by USDT volume;
  - immediate refresh on mount, and re-arm after a node reconnect (`resubscribeTickerBar` called from `nodeStore` after `CONNECTED`).
  - Verified live: a top pair's ticker timestamp and % change update in real time after page load and node switch.

### Changed
- Removed the **BitShares node settings** tab from the Settings page - node management is now fully handled by the Connection Settings modal (footer indicator / PIN screen).

## [1.2.0] - 2026-06-25

### Changed
- **Replaced the global pending-transaction firehose with targeted subscriptions.** Removed `set_pending_transaction_callback` (which streamed and parsed every transaction on the network). Real-time data is now addressed:
  - `subscribe_to_market(base, quote)` drives the order book + trade-history refresh for the **current pair only** (throttled), giving near-instant updates with minimal traffic and ~zero CPU spent on unrelated transactions.
  - The user account is subscribed via `get_full_accounts([id], subscribe)`; balance/own-order updates arrive through the single `set_subscribe_callback` (block `2.1.0` + account) and `walletStore.accountMonitor`.
- **Clean subscription lifecycle:** switching trading pairs unsubscribes the old market before subscribing the new one; leaving the terminal tears the subscription down (`onUnmounted`); after a node reconnect `nodeStore` re-establishes both the market (`marketStore.resubscribeMarket`) and account (`walletStore.resubscribeAccount`) subscriptions - no duplicates.
- Top ticker bar now refreshes on a lightweight 20s interval instead of riding the firehose.

### Added
- **Connection Settings modal** (`ConnectionModal.vue`) opened by clicking the footer connection indicator (`NodeStatus`) and from a button on the PIN / auth screen. Shows the active node + ping, a ping-sorted list of available nodes with one-click connect, manual re-check, and a Personal-nodes tab to add/remove custom WSS endpoints.
- Added `data-testid` hooks across the connection modal, footer node status, and auth screen for automated testing.

### Fixed
- Asset-load race: pair assets are now ensured (`_ensurePairAssets`) before fetching the order book on market change / reconnect, eliminating the recurring "asset not found" console errors.
- Personal nodes appear in the modal immediately (optimistic insert) instead of waiting for the next background ping cycle.

## [1.1.0] - 2026-06-25

### Changed
- **Rewrote the node connection manager (`nodeStore`) as an explicit finite state machine** with states `IDLE | CONNECTING | CONNECTED | RECONNECTING | ERROR`. `connectionStatus` is now a backward-compatible getter derived from the FSM, so all existing consumers keep working.
- **Reconnection is now owned by `nodeStore`** with exponential backoff + jitter (`1s → 30s`, capped) and round-robin node failover, replacing the previous fixed-interval logic that could hammer a node when `attempts == 0`.
- **Single, coordinated subscription source.** Object/block and pending-transaction subscriptions are re-established by `nodeStore` strictly after entering `CONNECTED`, removing the previous `set_subscribe_callback` conflict between `event.js` and the store that caused realtime updates to silently drop after a reconnect.
- **Node latency checks (`checkNodes`) moved to a non-blocking background task.** First connection now goes straight to the preferred (or first) node for an instant startup; pings are refreshed afterwards.

### Fixed
- **Stale connect promise after a drop.** Added `src/xbts/api/connection.ts` wrapper that resets the cached `connectPromise` before every (re)connect, so a reconnect can no longer resolve with a "successful" but dead socket.
- **Removed the harmful `disconnect()` before the first connect**, which globally disabled the underlying auto-reconnect for the whole session. Node switching / forced reconnects now perform an atomic socket reset instead.
- **Connection status detection** now correctly handles the array payload delivered by `event.js`, so socket `closed`/`error` events actually trigger a managed reconnect.
- Footer no longer flashes **"No connection"** on page refresh - the FSM starts in `CONNECTING`, showing a connecting state immediately.
- Filtered local/LAN nodes (e.g. `ws://localhost:8090`) shipped in the library's default node list out of the background probe to stop `connection refused` console spam for hosted users.

## [1.0.0] - 2026-06-25

### Added
- Initial import and setup of the XBTS DEX UI (Vue 3 + Vite) trading terminal for the BitShares blockchain.
- Dev server adapted to run on port 3000 behind the platform proxy.
- Lightweight footer "Loading…" status indicator replacing the intrusive full-screen connection overlay on startup.
