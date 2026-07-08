# XBTS DEX UI v2


## [2.9.0] - 2026-07-08 — Price providers (Binance) + AUTO start-price toggle

### Added
- **External price providers:** the Grid Bot's reference price can now come from an external exchange (Binance) instead of the on-chain Bitshares ticker. New `priceProviders.js` (pure fetch + symbol mapping) and `priceProviderStore.ts` (global default provider, per-pair overrides, manual exchange-symbol overrides, API-key storage). Binance prices are pulled from the CORS-friendly public domain `data-api.binance.vision` (no API key required for price data).
- **Bot settings modal (`BotSettingsModal.vue`):** opened via a new gear icon in the Control Console header. Lets the user pick the global price source, override the source or exchange symbol per pair, test the connection live, and store optional exchange API credentials (kept locally for the user's own build).
- **AUTO start-price toggle:** the START PRICE field's AUTO switch is fully wired again — when ON (and the bot is idle) the live market price is auto-mirrored into START PRICE (pair is never changed); turning it OFF frees the field for manual entry. Added the missing `bot.autoprice*` i18n keys.

### Changed
- `fetchCurrentPrice()` (botEngine.js) is now provider-aware — the single choke point routes BOTH bot setup and the live runtime through the selected provider, with transparent fallback to the on-chain Bitshares price when a pair isn't listed on the exchange (e.g. STH → 400 → on-chain).

### Fixed
- Restored the AUTO price behaviour the previous refactor had detached (`autosync` was referenced in the template but no longer bound in the setup scope, and the live ticker never fed START PRICE).

