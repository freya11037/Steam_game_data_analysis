# Steam Review Surge Analysis

---

## Research Question

> Are review surge events associated with changes in review scores for commercial Steam games?

Using a panel of 606,431 weekly observations across 6,355 commercial Steam games (2020–2024), this project detects review surges and estimates their association with `pos_share` (the percentage of positive reviews) using a two-way fixed effects regression framework.

**Central finding:** Surges *look* positive in raw data (+0.91pp) but are associated with *lower* review scores once game quality and time trends are controlled for (−0.65pp). The sign flip is the core result — it reflects a selection bias where surge periods attract a different reviewer composition, not necessarily more satisfied players.

> *"Surges don't signal momentum — they signal disruption."*

---

## Pipeline Overview

The analysis is structured as three sequential notebooks:

```
Stage 1 → Stage 2 → Stage 3
  Clean     Detect    Estimate
  Panel     Surges    Effects
```

| Stage | Notebook | Input | Output |
|---|---|---|---|
| 1 | `Stage1_DataPipeline.ipynb` | `games.json`, `aggregated_reviews_weekly.csv` | `panel_stage1_clean.csv` |
| 2 | `Stage2_SurgeDetection.ipynb` | `panel_stage1_clean.csv` | `panel_with_surges_weekly.csv` |
| 3 | `Stage3_PanelRegression.ipynb` | `panel_with_surges_weekly.csv` | Regression tables, findings |

---

## Stage 1 — Data Pipeline

Builds a clean weekly panel from two raw Mendeley data sources.

**Filters applied:**

| Filter | Value | Reason |
|---|---|---|
| Date window | 2020–2024 | Covers COVID, inflation, and baseline macro periods |
| Commercial only | `estimated_owners != '0 - 0'` | Removes demos, betas, and playtests |
| English only | `supported_languages` contains `'English'` | Standardises review language |
| Min lifetime reviews | ≥ 100 | Excludes games too sparse for a reliable baseline |
| Min weekly avg reviews | ≥ 3 | Eliminates near-zero-activity flatlines |
| Min active periods | ≥ 8 | Ensures enough history for the rolling window in Stage 2 |

**Output:** 606,431 observations, 6,355 games, 249 unique weeks. Median 86 weeks of observations per game. Mean `pos_share` across all game-weeks: 81.1%. Four macro-period dummies: `covid`, `post_covid`, `inflation`, `baseline`.

**Outcome variable:** `pos_share` — the share of positive reviews written in a given week. Interpreted as the weekly satisfaction signal. Selection effect acknowledged: surge-week reviews reflect *who chose to write at that time*, not a random sample of all owners.

---

## Stage 2 — Surge Detection

Flags surge weeks using a **rolling window baseline** rather than a global static threshold.

**Why rolling?** A static baseline (game lifetime mean ± 2 SD) is dominated by launch-week volume and mixes in launch effects driven by entirely different mechanisms (marketing, novelty). A rolling baseline compares each week against only the preceding N weeks, operationalising the idea that a surge is an *unexpected* deviation from recent norms.

**Hyperparameters:**

| Parameter | Value | Role |
|---|---|---|
| `WINDOW` | 8 weeks | Lookback period for the baseline |
| `MIN_PERIODS` | 8 weeks | Minimum history before a surge can be flagged |
| `MULTIPLIER` | 2.0 | Standard deviations above rolling mean to qualify |

**Key outputs:**
- Surge rate: 7.7% of active weeks (46,641 game-weeks)
- Intensity categories: `low`, `medium`, `large`, `extreme`
- 99,880 NaN warmup rows (expected — first 8 weeks per game have no baseline)
- Validated against Persona 5 Royal: surge alignment confirmed with Steam sales, Persona 3 Reload launch (Feb 2024), and franchise spillover events

---

## Stage 3 — Panel Regression

Estimates the association between surge events and `pos_share` using **two-way fixed effects (TWFE)**, absorbing both game-level quality (entity FE) and seasonal/macro trends (time FE).

**Models:**

| Model | Specification | Key finding |
|---|---|---|
| M1 | Surge binary (0/1) | −0.65pp (p<0.001) — sign flips from descriptive +0.91pp |
| M2 | Surge intensity categories | All negative (−0.51pp to −0.88pp); non-monotonic pattern |
| M3 | Macro-period interactions | COVID nearly triples effect (−1.47pp vs −0.54pp baseline) |
| M4 | Maturity heterogeneity | Mature games (≥6 months): −0.67pp significant; new games: −0.09pp not significant |
| M5 | Intensity × maturity | New games at large intensity: +0.91pp (p<0.001); mature games stay negative |

**Note on R-squared:** Within-R² of ~0.0002 is expected in two-way FE designs and is not a model quality concern. The estimator absorbs the vast majority of variation via fixed effects.

**Framing:** All results are reported as associations, not causal effects. The two-way FE controls for time-invariant confounders and common time trends, but without an instrumental variable or quasi-experimental design, causal claims are not warranted.

---

## Five Core Findings

1. **Sign flip** — Raw descriptive association is +0.91pp; after fixed effects it becomes −0.65pp, confirming selection bias in the unadjusted estimate.
2. **Non-monotonic intensity** — Medium surges are worst (−0.88pp); large surges partially recover (−0.48pp); extreme surges dip again (−0.78pp).
3. **COVID amplification** — During COVID, the surge effect nearly triples to −1.47pp (vs −0.54pp baseline). Inflation and post-COVID periods are not significantly different from baseline — this is specifically a COVID phenomenon, consistent with an influx of new casual players flooding the platform during lockdowns.
4. **Maturity concentration** — Surge effects are significant only in mature games (≥6 months post-release), where surges disrupt an established weekly reviewer rhythm.
5. **New game polarity flip** — At large intensity, new games show a *positive* association (+0.91pp), suggesting that high-attention launch-adjacent surges can attract enthusiastic early adopters.

---

## Theoretical Mechanisms

Three mechanisms motivated by the literature:

- **Player composition shift** — Surges attract players who do not reflect the core audience, altering `pos_share` independently of underlying game quality *(Li & Hitt, 2008; Schoenmueller et al., 2020)*
- **Expectation inflation** — Heightened visibility raises player expectations before play, making satisfaction harder to achieve *(Moe & Schweidel, 2012)*
- **Information cascade** — Surge attention amplifies both positive and negative sentiment signals, increasing review polarisation *(Muchnik, Aral & Taylor, 2013)*

---

## Data Source

Mendeley Data — Steam Reviews Dataset (2020–2024)  
Raw files: `games.json` (~65,000+ games metadata) + `aggregated_reviews_weekly.csv`

> **Note:** Raw data files are not included in this repository due to file size. Download directly from the Mendeley dataset page and place in your working directory before running Stage 1.


---

## How to Run

Run notebooks in order:

```
1. Stage1_DataPipeline.ipynb      → produces panel_stage1_clean.csv
2. Stage2_SurgeDetection.ipynb    → produces panel_with_surges_weekly.csv
3. Stage3_PanelRegression.ipynb   → produces regression outputs
```

Each notebook is self-contained with documented analytical decisions in the opening markdown cells.

---

## Repository Structure

```
Steam_game_data_analysis/
├── README.md
├── FinalReport.pdf
├── notebooks/
│   ├── Stage1_DataPipeline.ipynb
│   ├── Stage2_SurgeDetection.ipynb
│   └── Stage3_PanelRegression.ipynb
└── .gitignore
```

