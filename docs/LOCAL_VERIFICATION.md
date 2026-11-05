# Local verification

Checked November 5, 2026 using Python 3.12, Node 24, pnpm 11.25, PostgreSQL 16, and Docker Compose.

- Frozen uv and pnpm installs completed.
- Ruff, mypy, frontend formatting, TypeScript checking, and frontend build passed.
- All 28 Python tests passed against a separate disposable PostgreSQL database.
- Built the Docker API from this checkout and ran the documented January 2025 demo in fresh volumes: 100,000 selected source records, 95,783 accepted trips, and all 17 dbt data tests passed.
- The readiness endpoint and populated summary API returned HTTP 200.
- Browser walkthrough: populated overview and the last-seven-days filter, which returned 22,853 trips. No console errors were observed.
- Corrected the dashboard's project link to the copied repository.

Alternate ports and a separate Compose project were used to leave an existing local demo intact. The optional Airflow stack, three-month backfill, and full-scale throughput were not rerun during this check. Historical evidence elsewhere in the repository describes earlier runs.
