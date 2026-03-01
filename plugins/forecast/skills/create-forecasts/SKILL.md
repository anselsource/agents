---
description: Create demand forecasts with auditable artifacts from user-provided data files
---

In a sandbox get started with
```bash
uv run --with ansel-sh python forecast.py
```

# User-provided data and source files

ansel-sh auto-discovers CSV files, first from `./data` then from cwd. You may optionally specify a directory to discover files.

```python
from ansel_sh import forecasts

results = forecasts(data_dir="/path/to/data")
```

## Claude Cowork

User uploads land in `./uploads`. That's a good place to start.

## Reuse classified files

You may optionally reuse classified files across multiple forecasting runs.

```python
from ansel_sh import files, forecasts

classified = files()

one = forecasts(classified)
two = forecasts(classified)
```

# Auditable output artifacts

All output artifacts are written to `.ansel/<run_id>/` in the cwd. Each run gets an auto-incrementing ID. Artifacts you may find include:

- `files_manifest.csv` — classified input files
- `series_manifest.csv` — extracted time series
- `training_snapshot.csv` / `testing_snapshot.csv` — model training and testing data
- `model_instructions.md` — model configuration and instructions
- `target_snapshot.csv` — forecast target data
- `forecast_result.csv` — the final forecast output
