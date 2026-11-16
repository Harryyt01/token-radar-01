# Token Radar

Entry for the [token-radar-01](https://make.fun/c/token-radar-01) bounty on make.fun: *the best token launch radar*.

The radar scores every pre-token protocol in the sealed snapshot and ranks it by **implied value**: what a token would be worth if the protocol traded at the same market-cap-to-TVL multiple as the live-token protocols in its own category. The model is derived from the snapshot, not from a hardcoded list of names.

## How it works

`main.ts` (Node 22, standard library only) reads the sealed snapshot and writes a ranked board to `board.json`, a dashboard to `dashboard/`, and a dossier to `dossier.json`:

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

**Model — "radar-2", one function of the snapshot figures:**

```
implied = round( tvl_usd
  × category_multiple                     # median mcap/TVL of live peers in the category (≥3 peers, else the global median)
  × (1 + clamp(change_7d, −50, 150) / 100 × 0.5)
  × (0.6 + min(age_days, 1095) / 1095 × 0.4)   # age with no token; unknown age 365d
  × (1 + 0.02 × min(chains, 5)) )
```

**Board:** 226 rows — the 40 watchlist protocols (TVL ≥ $50M, no token) plus 186 pre-token protocols below that floor. The 24 live-token protocols are set aside as a comparison table; their market caps are facts, not a model.

**Farm plans:** 7 steps per protocol, parameterized by category, chain, TVL-tier capital and age-based hold days, each step with a cost and a time ("Action. Cost: …. Time: ….").

**Dashboard:** dark navy + amber, 3 files, search, category and chain filters, watchlist-only toggle, pre-token/live tabs, sortable columns, a per-protocol detail dialog with the model's own arithmetic and the protocol's farm plan, and a model explainer showing every sector multiple.

## Layout

- `main.ts` — entrypoint: reads the snapshot, writes board.json, data.js, the dashboard, the screenshots and dossier.json
- `lib/model.ts` — the model, the farm plans and the board issue check
- `vendor/makefun-board.ts` — the kit's library, copied in as its header instructs, used only to write dossier.json
- `dashboard/` — the page (index.html, app.css, app.js)
- `screenshots/` — board, detail and filtered views of the dashboard
- `input/` — not committed: the sealed snapshot the grader mounts at /input

## Verification

Run the entrypoint twice and diff the outputs: the same snapshot always writes the same bytes, so the grader's two runs agree. The board is checked locally against the kit's gate (100–4,000 rows, ranks 1..N in order, implied values non-increasing, 3–12 farm steps of 20–600 characters, every watchlist slug present, dashboard under the 64 KB dossier text cap).

Implied value is a model of what a token *could* be worth, not a price and not a prediction that a token is issued.
