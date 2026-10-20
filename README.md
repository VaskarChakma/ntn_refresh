# Which Satellite to Refresh Next?

**Look-ahead scheduling of orbit data for timing advance in LEO networks**

In 5G NTN, every UE computes its uplink timing advance from the satellite ephemeris before the first transmission. When the orbit copy is stale, the preamble lands outside the PRACH cyclic prefix and initial access fails. Suppose a network can refresh only *K* of its 1,000+ orbit copies every 10 minutes. Which ones should it pick?

This repository contains the full experiment behind our ICOIN 2027 paper. A scheduler learns how quickly timing error grows with data age, predicts which satellites are about to pass over the service area, and plans its refreshes with an exact Hungarian assignment that is re-solved every slot.

<p align="center">
  <img src="figures/F10_test_violation_vs_K.png" width="520" alt="Test-week violation rate versus refresh budget K">
</p>

## Key results

Real Starlink TLEs, 1,087 satellites over Seoul, PRACH format A3 tolerance (28.1 µs), evaluated on a held-out test week.

| Budget *K* (per 10 min) | Best baseline | Proposed | Gain |
|:---:|:---:|:---:|:---:|
| 5  | 0.170 | 0.165 | +3.4% |
| 8  | 0.071 | 0.063 | **+11.9%** (6/6 days) |
| 12 | 0.041 | 0.037 | **+10.6%** (5/6 days) |

- Under looser preamble formats, the gain reaches **23.8%** (C2) and **34.8%** (format 0).
- With 1 to 3 refreshes per slot, visibility-weighted heuristics perform about as well as the scheduler. We report this regime openly.

## Quick start

```bash
pip install -r requirements.txt
jupyter notebook ntn_ephemeris_refresh.ipynb   # Run All, about 5 min on a laptop
```

Everything is self-contained: no downloads and no API keys. Figures go to `figures/` and tables go to `tables/`.

## What the notebook does

| Blocks | Step |
|---|---|
| 1–3 | Setup, NR numerology (TS 38.211 CP lengths) and geometry, each followed by self-checks |
| 4 | Loads TLEs, splits train / validation / test and detects maneuvers from semi-major-axis jumps |
| 5–6 | Truth and visibility over Seoul on a 1-min grid, plus labelled (age, error) samples |
| 7 | Learns the monotone violation curve (isotonic regression with PAV) and checks its calibration |
| 8 | The scheduler and six baselines, including a Whittle-AoI index |
| 9 | Design selection on the **validation** window only |
| 10–12 | One-time **test** run, significance tests, ablations and tolerance sweep |
| 13 | Summary of 15 validity checks and all findings |

## Repository layout

```
ntn_ephemeris_refresh.ipynb   main experiment (13 blocks)
data/                         Starlink TLEs, 5-19 July 2022
figures/                      F07 violation curve, F10 main result, F11 ablation, F12 error CDF
tables/                       T00-T13: every number reported in the paper
requirements.txt              pinned versions (Python 3.11)
```

## Honest evaluation

- **No look-ahead.** The truth reference is the *next* published TLE, which no scheduler can ever hold.
- **Design fixed before testing.** All design choices are made on validation data, and the test week is used once.
- **Least favourable comparison.** Each budget is compared with the baseline that is best *on the test week itself*.
- **Statistics.** Paired daily bootstrap 95% CIs and one-sided Wilcoxon tests.

> **Reproducibility note:** scores can tie between satellites. Tie-breaking may differ across numpy/scipy versions and shift the third decimal. Use the pinned `requirements.txt` to reproduce the paper's numbers exactly.

## Data

Starlink TLEs from space-track.org (18th Space Defense Squadron), redistributed by the [Starlink autonomous driving dataset](https://github.com/self-driving-satellite-network/starlink-autonomous-driving) (Li et al., MobiCom 2023).

## Citation

```bibtex
@inproceedings{chakma2027refresh,
  title     = {Which Satellite to Refresh Next? Look-Ahead Scheduling of Orbit Data
               for Timing Advance in LEO Networks},
  author    = {Chakma, Vaskar and Choi, Wooyeol},
  booktitle = {Proc. International Conference on Information Networking (ICOIN)},
  year      = {2027}
}
```

**Contact:** Vaskar Chakma, Wooyeol Choi · Chung-Ang University, Seoul · {vaskar, wchoi}@cau.ac.kr
