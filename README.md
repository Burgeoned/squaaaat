# How Many People in the World Can Squat 315 / 405 / 495?

> **Superseded (July 2026):** the final harmonized model for all three lifts lives in
> [`../sbd_final`](../sbd_final). Squat numbers there match this model within rounding; the
> final version adds women's milestones (225/315/405), CIs on every per-sex count, and one
> shared engine across lifts. Note: this model's SBD-comparison reel slide (`reel_04b_sbd.png`)
> shows deadlift 405 = 4.38M, an older figure superseded by the deadlift model's own 5.09M.
> This README documents the model as originally published.

A data-driven estimate of how rare squatting 3, 4, and 5 plates actually is — across all 8.2 billion people on Earth.

The bench press is almost entirely a gym movement: you need a bench and a barbell, full stop. The squat is different — leg strength shows up in military training, combat sports, manual labor, and team sports, often without anyone ever touching a squat rack. So a naive "gym-goers only" model undercounts who can actually squat heavy. This project tries to count everyone who plausibly could, without double-counting anyone, and reports the result with honest uncertainty rather than a single made-up number.

---

## TL;DR results

| Milestone | Total worldwide | Men | Women | % of gym-goers |
|---|---|---|---|---|
| **315 lbs** (3 plates) | ≈5.43M | 1 in 442 | 1 in ~38,000 | 1.75% |
| **405 lbs** (4 plates) | ≈0.99M | 1 in 2,423 | 1 in ~384,000 | 0.32% |
| **495 lbs** (5 plates) | ≈0.19M | 1 in 12,803 | 1 in ~3.3M | 0.06% |

("Gym-goers" = the ~310M people worldwide estimated to do any resistance training, not just squatters.)

These are **medians from a 100,000-trial Monte Carlo simulation**, not single-point guesses — every number above has a 95% confidence interval in the notebook.

---

## Why this is hard to estimate honestly

Three problems undermine most "how many people can lift X" claims you see online:

1. **Using competition data directly.** The median OpenPowerlifting squatter trains specifically for meets and isn't representative of a normal gym-goer, let alone the general population.
2. **Ignoring who's outside the gym entirely.** Military personnel, combat sports athletes, and manual laborers can have serious squat-relevant leg strength without ever back squatting with a barbell.
3. **Picking one number instead of admitting uncertainty.** Lift-participation rates, regional strength baselines, and the shape of the strength distribution are all estimated from surveys and normative data — they have real error bars, and reporting a single confident-sounding number hides that.

This model addresses all three. Here's how, step by step.

---

## Step 1 — Real data, properly cleaned

[OpenPowerlifting](https://www.openpowerlifting.org/) is a public database of 3.9M+ recorded results from sanctioned powerlifting meets worldwide. We pull the squat column (`Best3SquatKg`), and apply three cleaning rules:

- **Raw or wraps only** — equipped (squat-suit) lifts are excluded; a suit can add 100+ lbs artificially and isn't a "raw human" number.
- **Plausible range** — anything under 45 lbs or over 950 lbs is a data-entry error, not a real lift.
- **Deduplicated to one lift per lifter** — the same competitor appears in the dataset many times across a career (the median lifter has ~2.8 entries). Without deduplication, prolific competitors — who tend to be stronger, since they keep competing — oversample the strong end of the distribution and inflate the apparent spread. We keep only each lifter's best recorded squat.

After cleaning: **332,367 unique male lifters** (median squat 419 lbs) and **148,961 unique female lifters** (median squat 237 lbs).

## Step 2 — Why we don't use those numbers directly

419 lbs is the median for *people who compete in sanctioned powerlifting meets* — a self-selected population that trained specifically to peak for a competition. That is nowhere close to "the average person," or even "the average gym-goer."

The model fits a **log-normal distribution** to the OPL data (strength can't go negative, and there's a long right tail of very strong outliers — this is the standard shape for physical performance data), then:

- **Shifts the median down** to normative strength data for casual gym-goers (NSCA percentile data, scaled by regional average bodyweight)
- **Tightens the spread (sigma)** — raw OPL sigma is inflated by elite competitors in the tail; we calibrate against NSCA trained-lifter percentile spreads instead, and sample sigma from a realistic range (0.28–0.38 for men, 0.29–0.40 for women) rather than using the OPL value directly

This single correction is the difference between "9 million people can bench 225" (the actual estimate from the companion bench-press model) and a much larger, unrealistic number you'd get by applying competition-lifter statistics to the general population.

## Step 3 — Three population pools, summed without double-counting

This is the core structural idea of the model. Instead of one population ("gym-goers"), there are three, and a person is only ever counted once:

### Pool A — Gym squatters
People in **10 world regions** who do resistance training *and specifically barbell back squat* (not every gym-goer does — many use machines, Smith squats, or skip legs entirely). Each region has its own:
- Resistance-training participation rate (sourced from IHRSA, CDC BRFSS, Eurobarometer, AusPlay)
- Fraction of trainers who back squat (lower for men than bench press fraction — squatting is less universally trained than benching; higher for women than their bench fraction — squat-rack culture has grown for lower-body training)
- Regional strength baseline, scaled to average bodyweight by region

### Pool B — Non-gym strength athletes
People with real squat-relevant leg strength who never specifically trained a barbell back squat: **military personnel, combat sports athletes (wrestling, judo, boxing, MMA), rugby and American/Canadian football players.** Two corrections keep this honest:

- **70% barbell-efficiency factor** — someone with strong legs from sport or military training can't fully express that strength on a barbell squat on day one (bracing, bar path, and depth under load all take reps to learn). Their effective squat number is discounted accordingly — conservative by design.
- **Overlap correction** — some fraction of these athletes *also* train in a gym and already barbell squat (already counted in Pool A). Regional overlap estimates range from 8% (Sub-Saharan Africa, limited gym infrastructure) to 45% (North America, widespread gym access). That fraction is subtracted from Pool B so nobody is counted twice. Net Pool B after correction: ~56M people (down from ~78M raw).

### Pool C — Youth athletes (age 14–15)
A small population of teenage athletes below the standard 16+ population floor (e.g. competitive youth powerlifters, serious high-school football programs). Included for completeness — adds on the order of **thousands** of people at 315 lbs, not millions. Real, but a rounding error against Pools A and B.

## Step 4 — Monte Carlo simulation, not a single guess

Every input above — lift rates, strength medians, distribution spread, Pool B size, overlap fractions — is an estimate with real uncertainty. Picking one "best guess" for each and reporting a single output number would be **false precision**.

Instead, the model runs **100,000 simulated trials.** In each trial, every uncertain parameter is independently redrawn from its plausible range (e.g., regional lift rate from a triangular distribution between its low/mode/high estimates; strength medians from a uniform range; sigma from a calibrated range). The full calculation runs once per trial, and the **median of the 100,000 outputs is the headline number; the 2.5th–97.5th percentile is the 95% confidence interval.**

This is why every number in this project comes with a range, not just a point estimate — e.g. 315 lbs: median 5.43M, 95% CI [3.21M – 8.20M].

## Step 5 — The denominator changes everything

"How rare is X" depends entirely on what you compare it to. The model reports against **four different denominators side by side**, so the framing is transparent instead of cherry-picked:

- **All 8.2 billion humans** — includes every infant, every 90-year-old, everyone with zero training access. The most honest "how rare on Earth" framing.
- **Able-bodied adults, age 16–70** (~4.67B) — removes people for whom training is physically off the table.
- **Men 16–70 only** (~2.37B) and **Women 16–70 only** (~2.30B) — since the male/female numbers tell very different stories (see below).
- **Gym-goers worldwide** (~310M, itself a Monte Carlo estimate) — the denominator most people who actually lift relate to: "of people who train, how many hit this?"

## Step 6 — Men vs. women

Squat strength requirements are the same in absolute terms regardless of sex, but training participation differs sharply, and the results reflect that:

| | 315 lbs | 405 lbs | 495 lbs |
|---|---|---|---|
| 1 in every ___ **men** | 442 | 2,423 | 12,803 |
| 1 in every ___ **women** | ~38,000 | ~384,000 | ~3,300,000 |

At 495 lbs, the model estimates roughly **690 women on Earth** can do it — smaller than the population of a small town. This isn't about strength potential; it reflects who specifically trains for maximal absolute strength and how.

---

## What would change the answer

If you think a number is wrong, here's exactly where to push back, because every input is a documented assumption rather than a hard fact:

- **Regional lift-participation rates** (the single biggest driver of uncertainty — see the tornado/sensitivity analysis in the notebook). North America and Europe's rates matter most because that's where most of the elite tail lives.
- **The sigma calibration** (0.28–0.38 men / 0.29–0.40 women) — using raw OPL sigma instead would inflate every estimate substantially, especially at 405+ lbs where the tail dominates.
- **Pool B's 70% barbell-efficiency factor** — a different assumption here directly scales Pool B's contribution.
- **Overlap correction fractions** — these are estimates, not measurements; the ±25% pool-size uncertainty in the simulation partially absorbs this.

All of these are visible, commented, and adjustable in the notebook — nothing is hardcoded without an explanation of where it came from.

---

## Files

- `squat_world_model.ipynb` — the full analysis notebook, runnable end to end, with inline commentary on every assumption
- `requirements.txt` — Python dependencies
- `*.png` — generated charts: standard landscape analysis figures plus 9:16 portrait slides formatted for short-form video

## Running it

```bash
pip install -r requirements.txt
jupyter notebook squat_world_model.ipynb
```

First run downloads the OpenPowerlifting dataset (~200MB) and caches it locally as `opl_data.csv` (gitignored — not included in this repo, regenerates automatically).

## Caveats

- Models *current capability*, not potential with training — someone could reach these numbers with enough work; this isn't a measure of who's capable in principle.
- "Squat" = raw, parallel-depth back squat specifically. Other variations (front squat, high-bar/low-bar, equipped) aren't modeled.
- Developing-region lift rates are the weakest inputs in the model — that's genuinely where most of the uncertainty lives, and the confidence intervals reflect it.
- 495 lbs is deep in the statistical tail of a fitted distribution; treat it as an order-of-magnitude estimate, not a precise count.
- The drop-off curve chart's Pool C (youth) contribution is computed slightly differently than the headline Monte Carlo totals — a sub-1% inconsistency, invisible at the precision shown in any chart, but worth knowing if you're auditing line by line.

See the notebook's appendix (final markdown cell) for the full regional assumptions table and data sourcing.
