# Aevum

A personal-finance app built around the **consumption tax** idea: overspending
relative to your own budgets imposes a self-billed tax, so you see the cost of
every discretionary rupee at the end of the week.

> "Aevum" is the product brand (`BRAND_NAME`); the legacy fallback name is
> "Personal Budget App", which is still this monorepo's directory name.

This is a monorepo with two **git submodules**, each its own repository:

| Submodule | Stack | Default port |
| --- | --- | --- |
| `backend/` | Python 3.13 · FastAPI · SQLAlchemy 2.0 (async) · **PostgreSQL** (asyncpg) · Alembic · Redis | 4000 |
| `frontend/` | React 18 · Vite | 5173 |

## Quick start

### Backend

Postgres + Redis run via docker-compose; the app + tooling run in a venv.

```bash
cd backend
docker compose up -d                                     # Postgres + Redis

python -m venv .venv
source .venv/bin/activate                                # or .venv\Scripts\activate on Windows
.venv/bin/pip install -r requirements.txt -r requirements-dev.txt

cp .env.example .env                                     # then fill in the REQUIRED vars
.venv/bin/python -m app.db.init_db                       # migrate + seed (also auto-runs on startup)
.venv/bin/python run.py                                  # dev server on http://localhost:4000
```

API docs: <http://localhost:4000/docs> (Swagger) and <http://localhost:4000/redoc>.
See [backend/README.md](backend/README.md) for the required env vars and the
full configuration reference.

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
│   └── docs/               # architecture / core / database / testing / per-module pages + archive/
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
  (PhonePe / Paytm UPI PDF, parsed via an async job pipeline + parser registry).
  Each `Transaction` carries a `TransactionCategorized` row per applied tag.
- **Budget limits** are per-(user, tag, period). Breaching one adds a *penalty*
  on top of the base consumption tax.
- **Weekly bill generation** runs on an incremental real-time ledger: every
  transaction mutation recalculates the week's tax, and a Monday worker
  finalizes the **ISO week** (Mon–Sun, in the user's timezone) through a
  5-state bill machine.

See [backend/docs/architecture.md](backend/docs/architecture.md) for the
full design and [backend/docs/database.md](backend/docs/database.md) for the
data model.

## Development

- **Tests**: `cd backend && .venv/bin/pytest` (runs against an ephemeral
  Postgres testcontainer; also `--cov=app` and `tests/benchmarks/benchmark.py`
  for the perf suite).
- **Lint**: `ruff` (the VS Code project is configured to run it on save;
  `E501` line-length is intentionally ignored).
- **Reset the dev DB**: `cd backend && scripts/reset-baseline.sh` (drop +
  recreate + migrate + seed). The dev database holds only dev/test data.

## Deploy

Target is **Render** (free tier to start) — both submodules deploy there. The
backend ships a Dockerfile + a `render.yaml` Blueprint and runs lean via
env-driven feature flags; see [backend/docs/deployment.md](backend/docs/deployment.md)
for the full runbook.
