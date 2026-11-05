# AtlasIQ

A data pipeline and dashboard for exploring NYC Yellow taxi trips. AtlasIQ takes the monthly TLC files through validation, PostgreSQL, and dbt before making the results available through FastAPI and a TypeScript dashboard.

I built the project around a practical question: how do you know the numbers in a chart came from a complete, traceable load? Source checksums, rejected-record storage, partition replacement, and publication checks make that process visible.

![AtlasIQ dashboard](portfolio/screenshots/overview.png)

[Case study](portfolio/case-study.md) · [Architecture](docs/ARCHITECTURE.md) · [Recorded verification](docs/VERIFICATION.md) · [CI runs](https://github.com/anusharama100/atlasiq/actions)

## What to look at

- **Ingestion:** monthly Parquet downloads, bounded retries, checksums, schema checks, and deterministic sampling.
- **Warehouse:** accepted and rejected records, source/run history, an incremental dbt fact table, dimensions, and analytical marts.
- **Publication:** newly loaded data stays out of the analytical API until transformations and quality checks finish.
- **Analysis:** demand by date, hour, and zone; fare and trip-efficiency comparisons; weather context; and twelve SQL analyses.
- **Interface:** seven dashboard views, date and zone filters, keyboard chart inspection, and CSV exports.

## Run locally

Install Docker with Compose, then:

```sh
git clone https://github.com/anusharama100/atlasiq.git
cd atlasiq
docker compose up -d postgres
docker compose build api
docker compose run --rm api atlasiq demo --month 2025-01 --limit 100000
docker compose up -d api
```

Open [localhost:8020](http://127.0.0.1:8020); API documentation is at [/docs](http://127.0.0.1:8020/docs).

The first run downloads the full January source file, about 59 MB, plus reference data, then selects 100,000 records. It needs network access. A fresh clone does not include source files or a populated database. PostgreSQL uses local port 5544; the optional Airflow UI uses 8082.

For a longer historical range:

```sh
docker compose exec api atlasiq backfill --start 2025-02 --end 2025-03 --limit 100000
```

See the [backfill guide](docs/MULTI_MONTH_BACKFILL.md) for partition replay and verification.

## How it is built

**Python, PostgreSQL, dbt, FastAPI, Airflow, TypeScript, SVG, and Docker.** The frontend has no runtime framework; compiled assets are included for the Docker path.

```text
TLC trips + zone lookup + NOAA observations + holiday calendar
  -> landed files and source checksums
  -> validation, accepted records, and quarantine
  -> dbt dimensions, facts, and analytical marts
  -> quality checks and publication
  -> API, dashboard, CSV, and SQL analyses
```

| Directory | Contents |
| --- | --- |
| `src/atlasiq/` | Ingestion, database, CLI, API, and compiled interface |
| `frontend/` | TypeScript dashboard source |
| `warehouse/` | dbt models, tests, and calendar seed |
| `orchestration/` | Airflow DAG and image |
| `analyses/` | Twelve SQL analyses |
| `tests/`, `docs/` | Tests, operational notes, and execution evidence |

Native development uses Python 3.12, uv, PostgreSQL, Node 24, and pnpm 11.25. Configuration and operational details are in the [runbook](docs/RUNBOOK.md) and [deployment guide](docs/DEPLOYMENT.md).

## Recorded results

The checked-in [three-month execution record](docs/evidence/multi-month-backfill.json) covers January–March 2025: 300,000 selected records, 284,899 accepted trips, and 15,101 rejections. February was replayed without changing partition counts or trip-ID fingerprints. Both combined builds passed all 17 dbt tests.

The [verification notes](docs/VERIFICATION.md) also record 28 Python tests, a complete five-task Airflow execution, and 12 desktop/mobile browser checks. These are recorded local runs; they are not a hosted-service uptime record or a fresh benchmark on every checkout.

## Boundaries

The equal monthly sample caps do not estimate city-wide demand growth. Weather comparisons are descriptive, and one station's daily readings are only a city-level proxy. Full-month throughput has not been benchmarked.

This is a local application. Hosting, authentication, tested backup restoration, external alerts, migrations, and serving-table swaps during rebuilds remain deployment work. Source publication and NOAA coverage must be checked before loading newer months.

Trip data comes from [NYC TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page); weather comes from [NOAA GSOD](https://www.ncei.noaa.gov/products/land-based-station/global-summary-of-the-day). See [sources and attribution](docs/SOURCES.md), [data definitions](docs/DATA_DICTIONARY.md), and [design decisions](docs/ADRS.md).
