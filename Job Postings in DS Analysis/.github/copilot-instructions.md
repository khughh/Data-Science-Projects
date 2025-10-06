## Repo purpose (short)
This workspace contains an exploratory Data Science project that analyzes recent data-science job postings (CSV + Jupyter notebook). The primary artifact is the notebook `Job Postings Data Set.ipynb` which depends on a small helper module `feature_engineering.py` and the CSV `data_science_job_posts_2025.csv`.

## What to edit and why
- Notebook: `Job Postings Data Set.ipynb` — main analysis, visualizations, and modeling. Prefer adding new cells or editing existing code cells (keep markdown narrative intact).
- Module: `feature_engineering.py` — authoritative data cleaning pipeline used by the notebook. If you change column names or cleaning behavior, update the notebook cells that call `FeatureEngineer` accordingly.
- Data: `data_science_job_posts_2025.csv` — raw data source. Do not modify in place; create copies when experimenting.

## Quick dev workflow (what actions an AI agent will commonly do)
- Run the notebook locally in VS Code or Jupyter. The notebook expects pandas and common ML libs (scikit-learn, seaborn, matplotlib, numpy).
- Typical first cell: `import pandas as pd` and `df = pd.read_csv("data_science_job_posts_2025.csv")` — ensure this path is correct relative to the notebook.
- Editing `feature_engineering.py`: class `FeatureEngineer(df)` exposes `engineer_all()` which returns the cleaned DataFrame. Use that API when creating tests or new pipeline variants.

## Key implementation details to preserve
- feature_engineering.FeatureEngineer:
  - engineer_all() calls: engineer_post_date, engineer_company_info, engineer_location, engineer_salary and then drops temporary columns `test`, `post_date`, `salary` before returning.
  - engineer_location() performs multi-step mapping: extracts location/headquarter, replaces On-site/Hybrid with headquarter, maps using `state_city_to_country` and `location_to_continent`. If adding new locations, update `_init_state_city_to_country` and `_init_location_to_continent` maps.
  - engineer_company_info() tries to reconcile `ownership`, `company_size`, and `revenue` fields and converts revenue strings (e.g., '€5M', '€2B') to numeric via convert_revenue_to_numeric.
  - engineer_salary() expects euro-formatted strings (e.g., "€1,000 - €2,000" or single value) and produces `min_salary`, `max_salary`, and `mean_salary` floats.

## Patterns and conventions
- Notebook-first development: analysis and model pipelines are implemented inline in the notebook rather than a separate package. When adding reusable logic, prefer adding to `feature_engineering.py` and keep the notebook calls minimal.
- Currency: salary and revenue parsing assume Euro formatting (prefix '€'). Conversions to USD or other currencies are done downstream in the notebook (not automated in the module).
- Location mapping is dictionary-driven. Add explicit city/state tokens to `_init_state_city_to_country` rather than complex geocoding for reproducibility.

## Testing and validation hints for AI edits
- Small, fast unit checks: import `feature_engineering.FeatureEngineer` in a lightweight script or cell, call `engineer_all()` on a sliced DataFrame (e.g., first 10 rows) and assert expected columns: `mean_salary`, `location`, `headquarter`, `company_size`, `revenue`.
- When changing parsing rules (salary/revenue), include a few example inputs in tests (e.g., '€1,200 - €2,400', '€3,000', '€5M') and assert numeric outputs.

## Files to reference when making changes
- `Job Postings Data Set.ipynb` — example usages and modeling pipelines (RandomForest, ElasticNet). Use it to follow current preprocessing and modeling choices.
- `feature_engineering.py` — canonical cleaning logic and mapping tables.
- `data_science_job_posts_2025.csv` — source data; open sample rows before changing parsing rules.

## When to ask for clarification
- If you need to change the currency assumption (currently euro), ask whether to convert inside the module or handle conversion in the notebook.
- If you plan to add geocoding or external API calls, get permission — the repo expects deterministic, offline transforms (dicts and string parsing).

If you'd like, I can also add a tiny test script and a short README that documents the minimal Python environment (pandas, scikit-learn, seaborn, matplotlib, numpy).
