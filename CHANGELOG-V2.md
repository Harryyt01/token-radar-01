# Changelog — Token Radar V2

## V2 (2026-11-16) — Live Crypto Launch Intelligence Dashboard

A new static app in `v2/` (no build step, no dependencies, no API keys). The V1
entry (`main.ts`, `lib/`, `vendor/`, `dashboard/`, screenshots) is untouched and
preserved; the V1 board snapshot is copied to `v2/data.js` and rendered inside
Explorer as the "Pre-Token Board", labelled as model data from 2026-11-14.

### Fixes (2026-11-16, after live browser verification)

- **GoPlus risk badges** — the live check found most Trending cards reading
  "Risk: check failed". Root cause: GoPlus's Solana `token_security` endpoint
  returns code 5000 "system error" for every Solana token (verified for USDC
  and WSOL too), so the check never ran. The badge now distinguishes three
  honest states instead of one misleading one: "Risk: unavailable" when
  GoPlus errors (check did not run), "Risk: no record" when GoPlus answers
  but holds no record for the token, "Risk: check failed" only for a real
  request failure. GoPlus calls are also queued (~1 per 400ms) so the badge
  loader can't burst past the free tier. The detail modal already said
  "Security check unavailable" — it now gets the cleaner message.
- **Radar label overlap** — labels in dense clusters overlapped. Labels are
  now drawn in a separate pass, largest bubbles first, skipping any label
  whose box would overlap one already drawn, with a dark halo for readability.

### New features

- **Live Radar** (`v2/js/radar.js`) — canvas radar with rotating sweep beam,
  bubbles from the real live trending feed (size = 24h volume, colour = 24h
  change, pulsing ring = launched < 1h ago), slow orbital drift, connecting
  lines to the 3 nearest neighbours, decorative particles, click/tap to open a
  token's detail, hover tooltip. Respects `prefers-reduced-motion` (static
  radar, no sweep). Single rAF loop.
- **Animated charts** (`v2/js/charts.js`) — canvas candles + volume bars from
  GeckoTerminal OHLCV, line-mode toggle, timeframes 1m/5m/15m/1h/4h/24h,
  crosshair + tooltip on hover/drag, refresh merges new candles instead of
  rebuilding, loading skeleton, explicit "no chart data" state.
- **Token discovery** (`v2/js/discovery.js`) — New Launches (GeckoTerminal new
  pools, newest first), Trending (GeckoTerminal trending pools + DexScreener
  boosts badge), Biggest Gainers / Biggest Losers / Highest Volume (computed
  client-side). Cards show logo, name, symbol, price, market cap/FDV,
  liquidity, 24h volume, 24h change, age, chain, contract address (click to
  copy) and a risk badge.
- **Token intelligence** (`v2/js/intelligence.js`) — detail modal (price,
  chart, liquidity, volume, age, transactions, contract) + GoPlus security
  report: verification, mint/freeze authority, proxy, owner, honeypot, buy/sell
  tax, pausable, blacklist, self-destruct, holder distribution with top-holder
  concentration, liquidity lock status. Risk score (Low/Moderate/High/Critical)
  with explicit warnings. **Never labels a token "Safe".**
- **Risk Scanner** — scan any contract address on 8 chains via GoPlus.
- **Watchlist** (`v2/js/watchlist.js`) — localStorage, add/remove from any card
  or detail panel, batched live price refresh on a timer, animated updates.
- **9 hash-routed views** — Overview, Live Radar, New Launches, Trending
  Tokens, Token Explorer, Market Analytics, Watchlist, Risk Scanner, Settings.
  Desktop sidebar, mobile bottom nav (< 860px).
- **Settings** — refresh interval (15s/30s/60s/off), chain filter,
  reduced-motion toggle, clear cache, clear watchlist.
- **Live activity toasts** — new launches detected on refresh.
- **Error boundary** — `window.onerror` shows a non-blocking banner; per-panel
  error states with a retry button.
- **Premium UI** (`v2/css/app.css`) — Apple-style dark glassmorphism, soft
  glows, rounded panels, system typography, skeleton loaders, mobile-first
  responsive, reduced-motion media query respected.

### Data sources (all free, keyless)

- DexScreener — pair search, token pairs, top boosts, watchlist refresh.
- GeckoTerminal — trending pools, new pools, OHLCV candles (timeframes mapped
  to `minute`/`hour`/`day` + `aggregate`, per the API's accepted values).
- GoPlus — token security (honeypot, taxes, mint, holders, lock status).
- Caching in-memory + localStorage with TTLs; GeckoTerminal queued to ~1
  request / 2s; 429 backoff with one retry; 12s timeout; every response is
  shape-checked before use.

### Data honesty

- Every panel labels its source and freshness ("DexScreener · 12s ago"); data
  older than 2 minutes is labelled stale.
- A failed API shows an error state with retry — stale data is never presented
  as live.
- Missing fields render as "—" or "Unavailable"; nothing is invented.
- The radar's ambient particles are decorative only and labelled as such.
- The V1 board is labelled "Snapshot 2026-11-14 — model data, not live prices".

### Known limits

- **GeckoTerminal free tier** is 30 requests/minute: trending + new pools poll
  3 networks each (eth, solana, base), queued at ~1 request / 2s, so a full
  refresh takes ~6s; results are cached 60s.
- **OHLCV** is only available for pools GeckoTerminal indexes; some new or
  low-liquidity pools return no candles ("No chart data" state).
- **GoPlus** covers Ethereum, BSC, Polygon, Arbitrum, Optimism, Base, Avalanche,
  Fantom, Cronos, Linea and Solana only; other chains show "chain not covered".
  GoPlus reports holder *lists* (top holders) but not a full distribution, and
  lock status only per top holder — a full lock/burn proof is not available
  from this source and is labelled "Unknown — not verified" when absent.
- **Holder data** is unavailable for many tokens/chains; the panel says so
  instead of inventing a distribution.
- **DexScreener search** returns at most 30 pairs per query; the Explorer caps
  results at 24 cards.
- The app is static and client-side: it needs network access to the three
  public APIs; with no network it shows error states, not data.
- `v2/data.js` is a copy of the V1 snapshot (2026-11-14); it does not update.

### Verification (2026-11-16)

- `node --check` passes on all 7 JS files in `v2/js/`.
- API checks (curl, live): DexScreener search 200 (30 pairs), DexScreener
  boosts 200 (30 items), GeckoTerminal trending_pools 200 (20 pools),
  new_pools 200 (20 pools), OHLCV hour/minute/day 200 (5 candles each),
  GoPlus token_security for USDT on eth code 1 with all fields present.
- Served with `python3 -m http.server`: every file in `v2/` returns 200 and
  `v2/index.html` references only files that exist.
- No API keys, secrets or localhost-only URLs anywhere in `v2/`.
- `git status --short` shows only `v2/`, `README.md` and `CHANGELOG-V2.md` as
  new/modified; all V1 files are untouched.

## V1 (2026-11-16) — Pre-Token Protocol Radar

- Original make.fun token-radar-01 entry: "radar-2" model ranking 226
  pre-token protocols by implied value from the DeFiLlama snapshot.
- Static dashboard in `dashboard/` (search, filters, sortable board, detail
  dialog with farm plans, model explainer).
- Verified: deterministic double-run (same snapshot writes the same bytes),
  board gate checks pass (coverage, ranks 1..N, non-increasing implied values,
  farm steps in range).
