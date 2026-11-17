# GTA VI Twitch CCU forecaster — v2

Forecasts GTA VI's launch-day Twitch viewers hour by hour (hours 2–24
UTC), five quantiles (5%, 25%, 50%, 75%, 95%) per hour.

## How it works

`main.py` (Python 3.13, standard library only, offline, deterministic)
fits, for each hour, a size-weighted regression of the log-ratio
`log(viewers_h / viewers_1)` on log hour-1 viewers across the case's
own comparables (past big launches), with:

- a **quadratic term in log hour-1** — the growth curve bends: small
  launches keep climbing into the evening, the biggest flatten early;
- a **Thursday dummy** — after size and curvature, Thursday releases
  sit ~0.5 log points below the line late in the day (GTA VI releases
  on a Thursday, 19 Nov 2026);
- 15% shrinkage toward the pooled median, spread at half the residual
  scale, slope smoothing across ±1 hour, and the house baseline as a
  fallback when too few comparables cover an hour.

## Validation

Practice loss **0.170** vs the house baseline's 0.387 (answers
reconstructed from the kit's cited SullyGnome source; all four hour-1
prints match the cases exactly). Leave-one-out over the comparables
improves over v1 on every cut: big launches (hour-1 > 30k) 0.270 →
0.240, very big (> 100k) 0.228 → 0.212, all 0.529 → 0.420,
Thursday launches 0.309 → 0.182.

## Run

```sh
MAKEFUN_INPUT_DIR=<dir with *.in> MAKEFUN_OUTPUT_DIR=<dir> python3 main.py
```

Reads every `*.in` case, writes `<id>.out` with 23 rows (hours 2–24),
each with five monotone quantiles in (0, 1e9].
