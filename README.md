# F1 Baku Winner Prediction

An XGBoost model that ranks the field for the Azerbaijan Grand Prix before the
race weekend starts — using only past race results, no qualifying, grid,
weather or live timing data.

**Latest run (2026-09-22) picks Andrea Kimi Antonelli, 53.6%.**

---

## Results

Trained on 2019–2024, tested on the full 2025 season it never saw.

| Method | Winners called | Races | Accuracy |
|---|---|---|---|
| XGBoost | 8 | 24 | 33.3% |
| Baseline (best recent average finish) | 7 | 24 | 29.2% |

Top 5 for Baku 2026:

| # | Driver | Team | Score |
|---|---|---|---|
| 1 | Andrea Kimi Antonelli | Mercedes | 53.6% |
| 2 | Max Verstappen | Red Bull | 13.2% |
| 3 | Lando Norris | McLaren | 10.9% |
| 4 | George Russell | Mercedes | 7.8% |
| 5 | Charles Leclerc | Ferrari | 5.5% |

Scores are normalized to sum to 100% across the field. They rank drivers — they
are not calibrated win probabilities.

---

## How it works

1. **Data** — every driver result from 2018 to now, pulled from the Ergast API
   through FastF1. 3,766 results across 187 completed races, through the
   Spanish GP on 2026-09-13.
2. **Features** — each row is one driver at one race, described by nine numbers
   built only from races that happened *before* it.
3. **Training** — XGBoost classifier, target is "did this driver win".
   Split by season so whole races stay together and nothing leaks forward.
4. **Testing** — score every driver in each 2025 race, take the highest, check
   it against the real winner.
5. **Prediction** — refit on all races before Baku, then score the current grid.

### The nine features

| Group | Feature | Meaning |
|---|---|---|
| Driver form | `driver_finish_5` | Average finish over last 5 starts |
| Driver form | `driver_points_5` | Average points over those starts |
| Driver form | `driver_win_rate_5` | Share of those starts won |
| Driver form | `driver_podium_rate_5` | Share finished top 3 |
| Team form | `team_points_5` | Average combined team points, last 5 races |
| Team form | `team_win_rate_5` | Share of those races won by either car |
| Circuit history | `circuit_finish_3` | Average finish at this track, last 3 visits |
| Circuit history | `circuit_points_3` | Average points at this track |
| Circuit history | `circuit_starts` | How many times they've raced here |

---

## Project structure

```
f1-baku-prediction/
├── notebooks/
│   ├── baku_model.ipynb               # the model, clean and runnable
│   └── baku_run_2026-09-22.ipynb      # the 2026-09-22 run, outputs and chart saved
└── outputs/
    ├── baku_model.joblib              # trained model
    ├── baku_model.json                # same model, XGBoost native format
    ├── baku_model_metadata.json       # features, versions, test metrics
    ├── baku_predictions.csv/.json     # full field, ranked
    ├── baku_inputs.json               # feature values fed to the model
    ├── Baku_XGBoost_Dashboard_Data.json
    └── baku_lovable_bundle.zip        # everything above, for the dashboard
```

---

## Running it

Python 3.10+, no GPU. Takes about two minutes, most of it downloading race data.

```bash
python3 -m venv .venv
./.venv/bin/pip install "fastf1>=3.6,<4" "scikit-learn>=1.6,<2" \
    "xgboost>=3.0,<4" matplotlib joblib nbconvert ipykernel
./.venv/bin/python -m nbconvert --to notebook --execute \
    notebooks/baku_model.ipynb
```

First run creates a `baku_cache/` folder (~4MB) of downloaded results, so later
runs are faster. It's gitignored.

To predict a different season, change `TARGET_YEAR` near the top. To override the
grid — substitutions, rookies — drop a `baku_entries.csv` next to the notebook
with `driverId, constructorId, driver, team` columns.

---

## Limitations

- **One race of edge.** 8 correct vs 7 for a one-line baseline, over 24 races.
  That gap is well inside noise. The model isn't clearly better yet.
- **Top-1 accuracy only.** No log loss, no calibration curve, so there's no
  evidence the 53.6% means anything as a probability.
- **Single test season.** One 2025 split, not rolling time-series CV.
- **Blind to the weekend.** No qualifying, grid slot, weather, tyre strategy or
  reliability — at a street circuit like Baku, where the safety car rewrites
  races, those matter more than form.

## Next steps

- Add qualifying gap, pit-stop deltas and teammate head-to-head as features
- Score with log loss and a calibration plot, not just top-1
- Rolling time-series CV across several seasons
- Track predictions against real results over a season to see if the edge is real

---


Data: [FastF1](https://github.com/theOehrly/Fast-F1) / Ergast.
