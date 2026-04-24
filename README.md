# practicex-backend

> FastAPI + PostgreSQL backend for [PracticeX](https://github.com/tylerj231/practicex-backend) — a self-directed programming practice platform.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [FastAPI](https://fastapi.tiangolo.com/) |
| Database | [PostgreSQL 15](https://www.postgresql.org/) |
| ORM / Migrations | [SQLAlchemy](https://www.sqlalchemy.org/) + [Alembic](https://alembic.sqlalchemy.org/) |
| Cache / Rate Limiting | [Redis 7](https://redis.io/) |
| Environment Manager | [Poetry](https://python-poetry.org/) |
| Auth | JWT (access + refresh tokens via `python-jose`) |
| AI | [Anthropic Claude](https://www.anthropic.com/) via `anthropic` SDK |
| Python | `^3.11` |

---

## Project Structure

```
practicex-backend/
├── app/
│   ├── api/
│   │   └── v1/              # Route handlers (one file per resource)
│   ├── core/
│   │   ├── config.py        # Pydantic Settings — reads from .env
│   │   ├── dependencies.py  # FastAPI dependencies (get_db, get_current_user)
│   │   └── security.py      # JWT helpers
│   ├── db/
│   │   ├── base.py          # SQLAlchemy declarative Base
│   │   └── session.py       # Engine + SessionLocal factory
│   ├── models/              # SQLAlchemy ORM models
│   ├── schemas/             # Pydantic v2 request/response schemas
│   ├── services/            # Business logic (auth, AI, progress, etc.)
│   └── main.py              # FastAPI app factory + middleware
├── alembic/                 # Alembic migration environment
├── tests/                   # Pytest test suite
├── scripts/                 # One-off scripts (seed data, etc.)
├── .env.example             # Required environment variables (copy to .env)
├── pyproject.toml           # Poetry project config + tool settings
├── Dockerfile               # Production multi-stage image
└── Dockerfile.dev           # Development image (hot-reload)
```

---

## Local Setup

### Prerequisites

- [Docker](https://www.docker.com/) and Docker Compose
- [Poetry](https://python-poetry.org/docs/#installation) (`curl -sSL https://install.python-poetry.org | python3 -`)

### 1. Clone and configure

```bash
git clone https://github.com/tylerj231/practicex-backend
cd practicex-backend
cp .env.example .env
# Edit .env and fill in real values (see Environment Variables below)
```

### 2. Start all services with Docker Compose

```bash
docker compose up
```

This starts:
- **backend** on `http://localhost:8000` (hot-reload enabled)
- **postgres** on `localhost:5432`
- **redis** on `localhost:6379`

### 3. Run database migrations

```bash
docker compose exec backend poetry run alembic upgrade head
```

### 4. Populate initial data

```bash
docker compose exec backend poetry run python -m scripts.seed
```

### 5. Verify

- API health check: `http://localhost:8000/health`
- Swagger UI: `http://localhost:8000/docs`

---

## Running Without Docker (bare metal)

```bash
poetry install
poetry run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Ensure `DATABASE_URL` and `REDIS_URL` in your `.env` point to running local instances.

---

## Environment Variables

Copy `.env.example` to `.env` and fill in all values before running.

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://postgres:postgres@localhost:5432/practicex` |
| `JWT_SECRET_KEY` | Secret used to sign JWTs (generate with `openssl rand -hex 32`) | `change_me` |
| `JWT_ALGORITHM` | JWT signing algorithm | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Access token lifetime in minutes | `15` |
| `REFRESH_TOKEN_EXPIRE_DAYS` | Refresh token lifetime in days | `7` |
| `REDIS_URL` | Redis connection string | `redis://localhost:6379` |
| `ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins | `http://localhost:5173` |
| `ANTHROPIC_API_KEY` | Anthropic API key for AI features | `sk-ant-...` |
| `ENVIRONMENT` | Runtime environment | `development` |

---

## Development

### Running tests

```bash
poetry run pytest tests/ -v
```

### Linting

```bash
poetry run ruff check .
```

### Type checking

```bash
poetry run mypy app/
```

### Creating a new migration

```bash
poetry run alembic revision --autogenerate -m "describe your change"
poetry run alembic upgrade head
```

### Rolling back a migration

```bash
poetry run alembic downgrade -1
```

---

## API Overview

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/api/v1/auth/register` | Register a new account |
| `POST` | `/api/v1/auth/login` | Login, receive JWT tokens |
| `POST` | `/api/v1/auth/refresh` | Silently refresh access token |
| `POST` | `/api/v1/auth/logout` | Clear refresh token cookie |
| `GET` | `/api/v1/topics` | List all topics |
| `GET` | `/api/v1/problems` | List problems (filterable) |
| `GET` | `/api/v1/problems/random` | Get a random problem |
| `POST` | `/api/v1/problems/generate` | AI-generate a new problem |
| `POST` | `/api/v1/sessions` | Start a practice session |
| `POST` | `/api/v1/sessions/{id}/run` | Run code against test cases |
| `POST` | `/api/v1/sessions/{id}/evaluate` | AI-evaluate a submission |
| `POST` | `/api/v1/sessions/{id}/hint` | Get an AI hint |
| `GET` | `/api/v1/progress` | Get progress summary |
| `GET` | `/api/v1/progress/heatmap` | 365-day activity heatmap |
| `POST` | `/api/v1/github/webhook` | GitHub webhook receiver |

Full interactive docs at `/docs` (Swagger UI) or `/redoc`.

---

## Branching Strategy

| Branch | Purpose |
|---|---|
| `main` | Production — protected, requires PR + passing CI |
| `develop` | Integration branch for feature work |
| `feature/*` | Individual feature branches, PR into `develop` |

---

## License

MIT © PracticeX