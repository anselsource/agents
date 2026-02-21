---
description: Create a demand forecast from a user-provided data file
---

To create a forecast:

1. Ask the user for the path to their data file (CSV or Excel)
2. Write a forecast script and run it with `uv run --with ansel-sh`:

```bash
uv run --with ansel-sh python forecast.py
```

Where `forecast.py` contains:

```python
import asyncio
from ansel_sh import File, create_forecast

file = File(uri="file:///path/to/data.csv", name="data.csv", created_by="user")
forecast = asyncio.run(create_forecast(files=file))
for item in forecast.items:
    print(f"{item.date}: {item.value}")
```

Replace the file path with the user's actual data file. No project setup or venv required — just `uv`.

The user must provide their own data file containing historical demand data.
