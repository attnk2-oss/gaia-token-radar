# Token Radar — gaia-token-radar

A token launch radar for make.fun's `token-radar-01` bounty: it reads a sealed
DefiLlama snapshot, ranks every pre-token protocol by what its launch would be
worth, and hands in a static dashboard with a farm plan for each one.

## Run

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22+, standard library only, no network. The program is deterministic: the
same snapshot always writes the same bytes.

## What it writes to /output

- `board.json` — 178 ranked rows (the 40-protocol watchlist + 138 early
  pre-token protocols below the $50M floor), each with an implied value and a
  7-step farm plan.
- `data.js` — the dashboard's dataset, beside the page.
- `dashboard/` — the static page (index.html, app.css, app.js; 45,531 bytes
  total, no network).
- `screenshots/` — three real headless-Chromium renders: the board, a
  protocol's detail view, a filtered view.
- `dossier.json` — built with `vendor/makefun-board.ts`.

## The model (radar-2)

```
implied = round( tvl_usd x 0.28
  x (1 + clamp(change_7d, -50, 100) / 100 x 0.6)
  x (0.55 + 0.45 x min(age_days, 1095) / 1095)
  x (1 + 0.02 x min(chains - 1, 5))
  x category )
```

TVL dominates; growth is clamped hard (one protocol in this snapshot grew
4,448% in a week); age is the time since DefiLlama first listed the protocol,
the best record of how long it has run with no token; breadth rewards
multi-chain distribution; `category` is a prior over what the protocol does
(lending and curators 1.05–1.10, bridges and stablecoin wrappers 0.65–0.70).
No protocol name is ever special-cased. The page recomputes every row from the
same figures and checks the number board.json carries, and a model sandbox in
the page re-ranks the board live when you move the dials.

## Coverage rule

The board ranks a protocol only when its record carries no token (no market
cap, no CoinGecko id, no symbol) and it is not a wrapped, staked or bridged
product of a token that already trades, not an exchange, and not a versioned
pool — the watchlist's own rule, applied to the early radar too. Live-token
protocols are set aside and shown with their market caps, never ranked.
"Records no token" is never written as "has no token": the record is not proof.

## Farm plans

Seven steps per protocol, derived from its own category, chains, TVL and age:
a fresh wallet, the category's action with a capital tier ($50–$250) and gas,
a weekly qualifying action, a hold window (30–60 days by age), a points-page
check, a never-pay-to-claim warning, and a farm log. Every step states its
cost and its time. The page derives the same seven steps, so what a reader
sees is what board.json carries (verified for all 178 rows in headless
Chromium; see `../.shot-tool/shoot.mjs`, kept outside this repo).

## Layout

- `main.ts` — entry point: read snapshot, score, write outputs, build dossier.
- `lib/model.ts` — the radar-2 model, the board builder, the farm plans.
- `vendor/makefun-board.ts` — make.fun's dossier library, byte for byte as
  shipped in the kit (not authored here).
- `dashboard/` — the page. `screenshots/` — the three renders.
- `out/` — local build output (not committed).

## License

MIT. See LICENSE.
