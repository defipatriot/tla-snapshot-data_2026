# tla-snapshot-data_2026

Hourly unified TLA snapshot. **This is the dashboard's primary data file.** Every TLA-registered pool with VP, depth, LP health, ampLP ratios, bribes, and per-bucket rollups.

Companion cron: [`defipatriot/cron-scripts/tla-snapshot`](https://github.com/defipatriot/cron-scripts/tree/main/tla-snapshot)

---

## Directory layout

```
tla-snapshot-data_2026/
├── README.md                          ← you are here
└── data/
    ├── tla-snapshot.json              ← latest hourly snapshot (the dashboard's read target)
    └── daily/
        ├── 2026-05-13.json            ← end-of-day archive (one per day, 23:xx UTC)
        └── ...
```

---

## Refresh cadence

Hourly at **:40 of every hour**, aligned with `network-and-prices` so the dashboard reads a consistent view of prices + pool state. Runtime ~30-60 seconds.

Output includes `nextRefreshExpectedAt` so the dashboard can show "next update in 47m" countdown.

---

## Schema (schemaVersion 1)

```jsonc
{
  "schemaVersion": 1,
  "capturedAt": "2026-05-13T12:16:48.079Z",
  "capturedAtUnix": 1778674608079,
  "refreshIntervalMs": 3600000,
  "refreshIntervalHours": 1,
  "nextRefreshExpectedAt": "2026-05-13T13:16:48.079Z",

  "epoch": {
    "currentEpoch": 184,
    "nextEpoch": 185,
    "epochStartedAt": "2026-05-11T00:00:00.000Z",
    "epochEndsAt":   "2026-05-18T00:00:00.000Z",
    "epochProgressPct": 35.9
  },

  "sources": {
    "network_and_prices": true,
    "bribes_current":     true,
    "bribes_history":     true,
    "votion":             true,
    "astroport":          true,
    "skeleton_swap":      true
  },

  "totals": {
    "tla_tvl_usd": 2388641,
    "depth_usd_total": 4562155,
    "active_pools_count": 33,
    "voted_pools_count": 33,
    "deprecated_pools_count": 0,
    "zero_vp_pools_count": 1,
    "total_pool_count": 67
  },

  "buckets": {
    "stable":   { "bucket_vp": 24108850e6, "bucket_vp_human": 24108850, "pool_count": 6,  "active_count": 4,  "tla_tvl_usd": 768820, "depth_usd": 928013 },
    "project":  { "bucket_vp": 23194129e6, "bucket_vp_human": 23194129, "pool_count": 19, "active_count": 8,  "tla_tvl_usd": 294792, "depth_usd": 1741872 },
    "bluechip": { "bucket_vp": 23141764e6, "bucket_vp_human": 23141764, "pool_count": 21, "active_count": 10, "tla_tvl_usd": 807042, "depth_usd": 1043892 },
    "single":   { "bucket_vp": 23464429e6, "bucket_vp_human": 23464429, "pool_count": 21, "active_count": 11, "tla_tvl_usd": 517988, "depth_usd": 848378 }
  },

  "pools": [
    {
      "name": "LUNA-USDC",
      "bucket": "stable",
      "dex": "Astroport",
      "dex_subtype": "concentrated",
      "pool_address": "terra1v3lqxl...",
      "lp_address": "terra1s275y73...",
      "is_lp_pair": true,
      "is_single": false,
      "source_type": "cw20",
      "status": "active",

      "voting_power": {
        "vp": 19726266042838,
        "vp_human": 19726266.04,
        "pct_of_bucket": 81.82,
        "lockup_contributions": [/* from votion data */],
        "votion_current_vp": 19725000,
        "votion_optimized_vp": 20000000
      },

      "depth_usd": 723306,
      "staked_in_tla_usd": 642912,

      "lp_health": {
        "asset_0": { "symbol": "LUNA", "amount_raw": 5201165825315, "amount_human": 5201165.83, "decimals": 6, "usd_value": 366500, "price_usd": 0.070465 },
        "asset_1": { "symbol": "USDC", "amount_raw": 360300294351,  "amount_human": 360300.29,  "decimals": 6, "usd_value": 360078, "price_usd": 0.999384 },
        "balance_ratio_pct": [50.44, 49.56],
        "total_pool_usd": 726578,
        "total_share": "1275204064684"
      },

      "amp_lp": {
        "underlying_lp_amount": 1128362899582,
        "shares": 1336718634096,
        "ratio": 0.844,
        "ratio_type": "non-amplified",     // > 1 = amplified, < 1 = fee-eroded
        "stake_mechanism": "astroport-incentives",
        "yearly_take_rate": 0.1
      },

      "sources": {
        "in_astroport_cron": true,
        "in_ss_cron": false,
        "in_staking_contract": true,
        "deprecated_in_astroport": null
      },

      "gauge_pool_id": "cw20:terra1s275y...",

      "bribes": {
        "active_now": [
          { "gauge": "stable", "asset": { "cw20": "..." }, "assets": [{ "info": { "native": "..." }, "amount": "19140721808" }] }
        ],
        "pd_historical_count": 0
      }
    }
    // ... 66 more pool entries
  ]
}
```

---

## The "active" rule

```
pool_pct_of_bucket = pool_vp / bucket_total_vp × 100

if pool_pct >= 1.0%:  status = "active"
elif pool_vp > 0:     status = "voted_but_below_threshold"
elif deprecated:      status = "deprecated"
else:                 status = "zero_vp"
```

This is the canonical rule TLA uses internally to determine which pools earn rewards. Verified empirically against the Eris Liquidity Hub UI's "Active" tab.

---

## Field reference

### `epoch`
Current and next TLA epoch numbers + start/end timestamps. The dashboard shows a countdown to `epochEndsAt` for the "Next vote ends in" timer.

### `totals`
Aggregate counts and USD sums across all pools.

- `tla_tvl_usd`: Sum of `staked_in_tla_usd` across all pools. Matches the "TVL" header on the Eris Liquidity Hub UI.
- `depth_usd_total`: Sum of DEX-side TVL (Astroport + SS pool depth). Generally higher than TLA TVL because not all LP tokens are staked into TLA.

### `buckets`
Per-bucket statistics. `bucket_vp_human` is the human-readable VP value (raw / 1e6).

### `pools[]`

Each entry includes:

- `status` — one of `active`, `voted_but_below_threshold`, `deprecated`, `zero_vp`. Dashboard typically renders only `active` by default, with a toggle to show `voted_but_below_threshold` for transparency.
- `voting_power.pct_of_bucket` — the 1% rule applies to this number.
- `lp_health` — present only for LP-pair pools (single-sided gauges like ampCAPA / xASTRO are null).
- `amp_lp.ratio_type`:
  - `amplified` (ratio > 1): rewards compound into the LP, shares appreciate
  - `non-amplified` (ratio < 1): take-rate erodes shares over time
  - `unity` (ratio = 1): newly created with no time elapsed
- `bribes.active_now` — raw active-bribe records keyed by LP token

---

## Cross-references

- `pool_address` matches `astroport-pool-data_2026.pools[].poolContract` and `ss-pool-data_2026.day-N.csv` `pool_address` column
- `voting_power.lockup_contributions` mirrors structure from `votion-data_2026.pools[].lockup_contributions`
- Active bribes mirror `bribes-data_2026.active_bribes` entries
- Token symbols + prices come from `network-and-prices-data_2026.token_prices`

---

## Known limitations (Phase A)

- Some cw20 LP tokens that aren't in the astroport-pool-data cron have unresolved names (shown as `cw20:terra1...`). Phase B will resolve these via `token_info` queries.
- Creda DEX pools (e.g. wBTC.creda.a) are not in standard chain `gauge_infos` and require a separate discovery mechanism. Currently missing from this snapshot.
- Rewards/APR/vAPR math not yet implemented (Phase B).
- `staked_in_tla_usd` for single-sided pools (ampCAPA, xASTRO) currently null because they don't have LP-pair health data. Phase B will compute via the single asset's USD price × staked amount.
