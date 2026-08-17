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

Column definitions and GeoMTS mapping hints are in [`manifest.yaml`](manifest.yaml).

## License

MIT — see [LICENSE](LICENSE). Synthetic / de-identified sample data only; no real PHI.
