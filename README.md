# Personal Budget App

A personal-finance app built around the **consumption tax** idea: overspending
relative to your own monthly budgets imposes a self-billed tax, so you see the
cost of every discretionary rupee at the end of the week.

This is a monorepo with two **git submodules**, each its own repository:

| Submodule | Stack | Default port |
| --- | --- | --- |
| `backend/` | Python 3.13 · FastAPI · SQLAlchemy 2.0 (async) · SQLite | 4000 |
| `frontend/` | React 18 · Vite | 5173 |

## Quick start

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate                                # or .venv\Scripts\activate on Windows
.venv/bin/pip install -r requirements.txt -r requirements-dev.txt

.venv/bin/python -m app.db.init_db                       # create & seed data.db (also auto-runs on startup)
.venv/bin/python run.py                                  # dev server on http://localhost:4000
```

API docs: <http://localhost:4000/docs> (Swagger) and <http://localhost:4000/redoc>.

### Frontend

```bash
cd frontend
npm install
npm run dev                                              # Vite dev server on http://localhost:5173
```

Override the API base with `VITE_API_URL` if the backend isn't on the default
port.

## What's where

```text
.
├── backend/                # FastAPI app — see backend/README.md
│   ├── CONTRIBUTING.md     # architecture rules & coding conventions
│   └── docs/               # architecture / core / database / testing / performance / per-module pages + refactor/
└── frontend/               # React SPA — see frontend/README.md
```

## Key concepts

- **Tags** are hierarchical and typed (`income` / `committed` / `essential` /
  `discretionary` / `exempted` / system-only `total` / `uncategorized` /
  `consumption_tax`).
- **Beneficiaries** specialise into `Merchant` or `Person`. The
  `CategorizationRule` table maps `beneficiary_id → tag_ids`; the engine
  applies the rules and propagates each leaf tag up its parent chain.
- **Transactions** can be manually entered or imported from a bank statement
  (PhonePe PDF / generic CSV). Each `Transaction` carries a
  `TransactionCategorized` row per applied tag.
- **Budget limits** are per-(user, tag, monthly period). Breaching one adds a
  *penalty* on top of the base consumption tax.
- **Weekly bill generation** (Sun–Sat) aggregates the week's debits, applies
  the tax rate for each `txn_type`, adds penalties for breached budgets, and
  writes a `CommitteeRevenueAcc` row plus per-txn `ConsumptionTaxTxn` rows.

See [backend/docs/architecture.md](backend/docs/architecture.md) for the
full design and [backend/docs/database.md](backend/docs/database.md) for the
data model.

## Development

- **Tests**: `cd backend && .venv/bin/pytest` (also: `--cov=app`,
  `tests/benchmarks/benchmark.py` for the perf suite).
- **Lint**: `ruff` (the VS Code project is configured to run it on save;
  `E501` line-length is intentionally ignored).
- **Reset the dev DB**: `rm backend/data.db && .venv/bin/python -m app.db.init_db`.
  `data.db` holds only dev/test data — safe to delete.
