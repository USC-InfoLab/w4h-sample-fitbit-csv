# w4h-sample-fitbit-csv

Synthetic **Fitbit CSV** sample package for the W4H Toolkit (data + `manifest.yaml` only). Import tooling lives in the paired repo **[w4h-sample-fitbit-csv-importer](https://github.com/USC-InfoLab/w4h-sample-fitbit-csv-importer)**.

**Publisher:** usc-infolab · Naming: `w4h-sample-<vendor>-<format>` ([ADR-016](https://github.com/USC-InfoLab/w4h-docs/blob/main/dev/ADR-016-paired-sample-importer-repos.md)).

## How to use

1. Clone this repo next to `w4h-sample-fitbit-csv-importer`.
2. Open **Contributions → Fitbit CSV importer** in the W4H app (or this repo’s sibling README) for notebook and CLI steps.
3. **Admins** ingest into their team catalog with a personal API key. **Users** use the importer notebook against data they already have access to; do not re-ingest **W4H Samples**.

## Contents

| File | Description |
|------|-------------|
| `data/fitbit_subjects.csv` | Subject id and study window (`start_date`, `end_date`) |
| `data/fitbit_weight.csv` | Weight, BMI, body fat — [Fitbit Get Weight Log](https://dev.fitbit.com/build/reference/web-api/body/get-weight-log/) |
| `data/fitbit_heart_rate.csv` | Intraday heart rate (bpm) — [Fitbit Get Heart Rate Intraday](https://dev.fitbit.com/build/reference/web-api/intraday/get-heartrate-intraday-by-date/) |
| `data/fitbit_calories.csv` | Intraday calories burned + METs — [Fitbit Get Activity Intraday](https://dev.fitbit.com/build/reference/web-api/intraday/get-activity-intraday-by-date/) (`resource=calories`) |

Column definitions, units, sampling intervals, and GeoMTS mapping hints are in [`manifest.yaml`](manifest.yaml) (see its `references` block for the Fitbit units/localization and intraday-resolution docs this sample follows). On import, signal and column descriptions are stored as PostgreSQL table and column comments. Sampling interval and value units are stored on `w4h.tables.mapping_json`.

## Known data-quality fixtures

This sample deliberately contains two realistic data-quality problems, left in the CSVs on purpose (not flagged in `manifest.yaml` — that file is the vendor mapping used to ingest and document the data, not an answer key). They exist so ingest/analysis tooling has something real to catch:

- **Outlier** — `data/fitbit_weight.csv`, subject `02f77d2`, `2018-11-09`: `weight=7.6` / `bmi=2.48`, versus ~73–77 kg for the same subject in the surrounding week. Reads as a manual-entry decimal slip. This is a statistical anomaly (the value parses fine as a valid `double precision`), so it can only be caught by analysis that looks across a subject's/cohort's other rows — not by ingest-time validation.
- **Tampered timestamp** — `data/fitbit_heart_rate.csv`, subject `02f77d2`, `2018-07-12`, row `15:06:37`: every other row in that intraday series lands on `:00` seconds, matching the signal's declared `sampling_interval: 1min`. This one row doesn't, simulating a hand-edited or corrupted export that no longer matches vendor spec. Unlike the outlier above, this is a structural mismatch the importer can and does check for at import time (see the paired [importer repo](https://github.com/USC-InfoLab/w4h-sample-fitbit-csv-importer)'s vendor-spec report).

Do not "fix" either row — if you add a new deliberate fixture, document it here, not in `manifest.yaml`.

## License

MIT — see [LICENSE](LICENSE). Synthetic / de-identified sample data only; no real PHI.
