# How Many People in the World Can Squat 315 / 405 / 495?

A data-driven estimate of how rare squatting 3, 4, and 5 plates actually is — across all 8.2 billion people on Earth.

## Method

1. **Data**: 3.9M+ real competition lifts from [OpenPowerlifting](https://www.openpowerlifting.org/), deduplicated to one best squat per unique lifter.
2. **Distribution fit**: Log-normal fit to raw (no suit/wraps-only) squat data, separately for men and women.
3. **Competitor → gym-goer correction**: OPL competitors have trained and peaked for a sanctioned meet. The model shifts the median down to normative gym-goer strength while keeping a calibrated spread (sigma).
4. **Two population pools, no double-counting**:
   - **Pool A** — gym-goers who specifically barbell back squat, broken out across 10 world regions with region-specific lift rates and strength baselines.
   - **Pool B** — non-gym strength athletes (military, combat sports, rugby/football) who have real leg strength without ever touching a squat rack. A 70% barbell-efficiency factor accounts for the gap between raw strength and untrained barbell technique. An overlap correction removes the fraction who *also* train in a gym (already counted in Pool A).
   - **Pool C** — youth athletes age 14–15 (e.g. high-school football, junior powerlifting), included for completeness since the standard 16–70 population base excludes them.
5. **Monte Carlo**: 100,000 trials sampling every uncertain parameter (lift rates, strength medians, sigma, pool sizes) from its plausible range to produce honest 95% confidence intervals instead of false precision.
6. **Denominator analysis**: results expressed against four different population bases — all 8.2B humans, able-bodied adults (16–70), men only, and women only — since the "how rare" framing changes a lot depending on what you compare to.

## Files

- `squat_world_model.ipynb` — the full analysis notebook, runnable end to end
- `requirements.txt` — Python dependencies
- `*.png` — generated charts (standard analysis figures + 9:16 portrait slides formatted for short-form video)

## Running it

```bash
pip install -r requirements.txt
jupyter notebook squat_world_model.ipynb
```

First run downloads the OpenPowerlifting dataset (~200MB) and caches it locally as `opl_data.csv` (gitignored — not included in this repo).

## Caveats

- Models *current capability*, not potential with training.
- "Squat" = raw, parallel-depth back squat. Equipped/suited squats and other variations are excluded.
- Developing-region lift rates are the weakest inputs in the model — that's where most of the uncertainty lives.
- 495 lbs is deep in the statistical tail; treat it as an order-of-magnitude estimate, not a precise count.

See the notebook's appendix for the full assumptions table and sourcing.
