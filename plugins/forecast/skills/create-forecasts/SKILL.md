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
