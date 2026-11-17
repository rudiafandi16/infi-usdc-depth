# INFI-USDC pool depth — auto-refreshed to Dune

Reproduces Dune query 7685210 (hourly / N-minute pool depth) straight from a Base RPC and
uploads the CSV to Dune every day via GitHub Actions.

- `main.go` — the pipeline (see `CLAUDE.md` for flags and semantics)
- `state.go` — incremental mode: checkpoint + mint/burn snapshot so each run only fetches new blocks
- `upload_dune.py` — pushes a CSV to Dune's table upload API (replaces the table)
- `.github/workflows/refresh.yml` — daily 03:00 UTC run for INFI-USDC (0xedb67e8e9f77b8e48a4ff81b376699d5c8ba2c98, USDC = token0) at 15-minute buckets
- `state/` — incremental checkpoint, created by the first run (delete it to force a full rescan)

## Pool

| | |
|---|---|
| Pool | `0xedb67e8e9f77b8e48a4ff81b376699d5c8ba2c98` (Aerodrome Slipstream, factory `0x9cb220581b91b0c83223012fe69688b92ad07a76`) |
| Tokens | USDC (token0, 6 dec) / INFI `0x9f4c76f421f47b3c234f841f477897f1d895177d` (token1, 18 dec) |
| Tick spacing / fee | 100 / 10000 |
| Deployed | 2026-11-16 05:34:17 UTC (block 58,084,411) |
| Trading | a few $0.04–0.10 test buys at $1.153 on 2026-11-17 07:28–09:15 UTC; real trading from 15:15 UTC |

The pre-launch test buys are real on-chain swaps, so the first buckets (07:15, 08:45, 09:00, 09:15)
appear with an almost flat price — filter `hour >= '2026-11-17T15:15'` if you only want live trading.

Data is fetched from `mainnet.base.org` with `-filter-logs` (that endpoint refuses `eth_getLogs`), and
block times follow Base's post-Denim 200 ms schedule; see `CLAUDE.md`. Verified locally on 2026-11-17:
the liquidity the script rebuilds from Mint/Burn equals the pool's on-chain `liquidity()` exactly.

## Setup

1. Repo → Settings → Secrets and variables → Actions → **New repository secret**:
   `DUNE_API_KEY` = your Dune API key (Dune → Settings → API).
2. Actions tab → "Refresh INFI-USDC depth" → **Run workflow** (or wait for the daily schedule).
3. Query it on Dune:

```sql
SELECT date_parse(hour, '%Y-%m-%dT%H:%i') AS ts, *   -- Dune stores hour as text like '2026-11-17T15:15'
FROM dune.forcemulti.dataset_infi_usdc_depth_15m
ORDER BY 1
```
