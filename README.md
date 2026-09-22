# F1 Baku Prediction — reference copy

Not my work. This is a downloaded Colab tutorial that predicts the Azerbaijan GP
winner with XGBoost. Keeping it here as reference, not as a portfolio project.
Do not push this as-is.

## What's here

- `notebooks/baku_tutorial_original.ipynb` — the tutorial, untouched
- `notebooks/baku_run_2026-09-22.ipynb` — same notebook after I ran it locally
- `outputs/` — model, predictions, dashboard JSON from that run

## How it works

Pulls F1 results 2018-2026 from FastF1/Ergast, builds 9 features (driver form over
last 5 races, team form, circuit history), trains XGBoost to predict the winner.
Trains on 2019-2024, tests on 2025, then refits on everything for the Baku pick.

## Run results (2026-09-22)

- 187 races of data, through the Spanish GP on 2026-09-13
- Test on 2025: 8/24 winners correct (33%)
- Baseline (just pick best recent average finish): 7/24 (29%)
- Baku pick: Antonelli 53.6%, then Verstappen 13.2%, Norris 10.9%

The win scores are normalized model scores, not real probabilities. And one extra
race over the baseline on 24 samples is noise, not a result.

## To run it

Needs python 3.10+, no GPU. Takes about 2 min.

    python3 -m venv .venv
    ./.venv/bin/pip install "fastf1>=3.6,<4" "scikit-learn>=1.6,<2" "xgboost>=3.0,<4" matplotlib joblib nbconvert ipykernel
    ./.venv/bin/python -m nbconvert --to notebook --execute notebooks/baku_tutorial_original.ipynb

Writes a `baku_cache/` folder (~4MB of downloaded race data) next to wherever you run it.

## If I build my own version later

Things the tutorial skips that would make it an actual project:

- Qualifying gap, pit stop deltas, teammate head-to-head as features
- Log loss and a calibration curve, not just top-1 accuracy
- Rolling time-series CV instead of a single test season
- Credit the tutorial as the starting point in the README
