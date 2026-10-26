# AGENTS.md — w4h-sample-fitbit-csv

Synthetic **Fitbit CSV** sample package: `data/*.csv` and `manifest.yaml` only. No import code. Column definitions, units, and the deliberate data-quality fixtures are in [README.md](./README.md) and [`manifest.yaml`](manifest.yaml). The shared W4H architecture overview is at the end of this file.

## Where this repo fits in W4H

- **Layer:** contribution, `kind: dataset`, catalog slug `w4h-sample-fitbit-csv`. The sample half of an [ADR-016](https://github.com/USC-InfoLab/w4h-docs/blob/main/dev/ADR-016-paired-sample-importer-repos.md) pair. Notebook, CLI, `run.sh`, and tests live in `w4h-sample-fitbit-csv-importer`.
- **Consumers:** the paired importer reads `../w4h-sample-fitbit-csv` (override `W4H_SAMPLE_FITBIT_CSV_PATH`). `w4h-api` clones this repo next to the importer before an in-app run. `w4h-docker` does not bake these CSVs into an image.
- **Signal slugs become table names:** `ds_<dataset_id>.fitbit_<slug>`, with mappings in `w4h.tables`. The prefix `fitbit` is shared with `w4h-fitbit-csv-file-importer`. `w4h-sql-compute-mets` and `w4h-demo-approximate-queries` bind those names.
- **GeoMTS:** no signal has a `geo` role. Metric units throughout (kg, kcal, bpm). `heart_rate` and `calories` declare `sampling_interval: 1min`. `weight` is a body-log, one row per entry, not an intraday series. `subjects` has no time series.

## Contents

| Path | What it is |
|---|---|
| `manifest.yaml` | Vendor mapping the importer posts: columns, units, `geomts_mapping`, `sampling_interval`. Descriptions become Postgres comments. This file is not an answer key. |
| `data/fitbit_subjects.csv` | Subject id and study window (`start_date`, `end_date`) |
| `data/fitbit_weight.csv` | Weight, BMI, body fat |
| `data/fitbit_heart_rate.csv` | Intraday heart rate (bpm), 1min |
| `data/fitbit_calories.csv` | Intraday calories (kcal) and METs, 1min |

`fitbit_calories.csv` is about 58 MB and `fitbit_heart_rate.csv` about 5 MB. Inspect headers and specific rows. Do not load either file in full.

## Change rules

| Change | Also update |
|---|---|
| CSV header, column, or new signal | Matching `manifest.yaml` entry (`columns`, `geomts_mapping`, `file`). Importer dedupe keys in `csv_import.py` (`subjects`: `id`; `weight`: `id,date,time`; `heart_rate`: `id,date,timestamp`; `calories`: `id,date,time`) and its tests. |
| `sampling_interval`, or units on `calories` / `weight` | Re-check `w4h-sql-compute-mets`. |
| A new deliberate bad row | [README.md](./README.md) section **Known data-quality fixtures** only. Leave it out of `manifest.yaml`. |

Keep CSV headers, value types, and manifest column names aligned. The importer reads the header with `csv.DictReader` and posts rows in 500-row chunks.

## Hard boundaries

- No application code, notebooks, or tests in this repo.
- No API keys, database credentials, or `.env` files. Import uses the paired importer; `W4H_API_KEY` stays in that repo's local `.env` or the process environment.
- Synthetic, de-identified sample data only. Do not commit real Fitbit exports or participant data.
- Leave the two documented fixtures in place. The importer reports the timestamp misfit and still imports the row.
  - **Outlier:** `data/fitbit_weight.csv`, subject `02f77d2`, `2018-11-09`, `weight=7.6`, `bmi=2.48`.
  - **Off-grid timestamp:** `data/fitbit_heart_rate.csv`, subject `02f77d2`, `2018-07-12`, `15:06:37` (every other row in that series is on `:00` seconds).

## Git expectations for agents

- Do not `git commit` or `git push` unless the user asks.
- Coordinate manifest or CSV shape changes with `w4h-sample-fitbit-csv-importer`.

<!-- BEGIN w4h-system-overview: identical in every W4H repo's AGENTS.md. Edit all copies together. -->

## W4H system overview (shared across all repos)

**W4H (Wearables for Health) Toolkit** loads wearable-device data into PostgreSQL, organizes it by team, project and dataset, controls access, and runs curated analyses on it. Each signal is a **GeoMTS** (Geospatial Multivariate Time Series) table whose roles (`subject`, `time`, optional `geo`, `values[]`) are stored in `w4h.tables.mapping_json`, not in fixed column names. Three repos form the platform, one packages it, and the rest are **contributions** listed in the in-app catalog (`w4h.contributions`).

| Repo | Role |
|---|---|
| `w4h-api` | Express API and Postgres DDL (`w4h` schema): auth, ACL, ingest, catalog, contribution runner, SQL engine. Port 2026 |
| `w4h-app` | Vue 3 + Vite SPA; talks only to `w4h-api`. Port 5173 |
| `w4h-docs` | Product rules, MVP, ADRs and guides; imported by the API as in-app Help. Owns product decisions |
| `w4h-docker` | Compose stack (`api`, `app`, `notebooks`) built from sibling repos; Postgres is external |
| `w4h-sample-fitbit-csv`, `w4h-sample-icde-demo-csv` | Synthetic sample CSVs plus `manifest.yaml` (ICDE includes a `gps_track` geo signal) |
| `w4h-sample-fitbit-csv-importer`, `w4h-sample-icde-demo-csv-importer` | Runnable importers (notebook, CLI, `run.sh`) for those samples |
| `w4h-fitbit-csv-file-importer` | Runnable importer for user-supplied Fitbit CSVs: profile headers, review a drafted mapping, ingest |
| `w4h-sql-compute-mets` | SQL contribution (YAML + SQL template) that `w4h-api` executes in-process |
| `fft_approximate` | Postgres C extension, catalog slug `fft-approximate`; no API involvement |
| `w4h-demo-approximate-queries` | Notebook comparing `fft_approximate` with exact averages over direct Postgres |

**Main flows:**

1. **Ingest:** importers post 500-row chunks to `POST /datasets/:id/import/csv-batch`. `lib/ingest-csv.js` creates `ds_<dataset_id>.<table_prefix>_<signal>` and upserts its `w4h.tables` mapping.
2. **In-app Run (ADR-022):** `POST /contributions/:slug/run` makes `lib/contribution-runner.js` spawn `../<slug>/run.sh` with the user's `W4H_API_KEY`, `W4H_API_BASE` and `W4H_DATASET_ID`. Stdout lines `@@PROGRESS@@ {json}` drive the progress bar.
3. **SQL analytics (ADR-023):** `lib/sql-contribution-engine.js` reads `../<slug>/manifest.yaml` and `sql/`, binds identifiers from `w4h.tables`, and runs in-process. There is no free-SQL endpoint.
4. **Extensions:** the operator installs `fft_approximate` on their Postgres; notebooks call it directly.

**Layout:** clone every repo as a sibling in one folder. The API finds contributions at `../<slug>` (override `CONTRIBUTION_RUNNER_REPO_ROOT`) and docs at `DOCS_REPO_PATH=../w4h-docs`.

**Cross-repo changes:** `w4h-api` first (regenerate `API_INDEX.md`), then `w4h-app`, then `w4h-docs`. Catalog rows are seeded in `w4h-api/data/schema.sql`. Start at `w4h-docs/dev/mvp.md` and `w4h-docs/dev/ADR-README.md` for decisions, and `w4h-docs/dev/developer-setup.md` for local setup, getting an API key, test personas, troubleshooting, per-repo test commands, default branches and a glossary. Do not commit or push unless asked; never commit `.env` files, API keys or participant data.

<!-- END w4h-system-overview -->
