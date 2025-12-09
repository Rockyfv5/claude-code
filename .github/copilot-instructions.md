### Purpose

This file gives focused, repo-specific guidance to AI coding agents so they become productive quickly in the
Platziflix codebase. It points to the big-picture architecture, key developer workflows, and project conventions
that are repeatedly used across the repository.

### Quick Architecture Summary

- **Backend (FastAPI + SQLAlchemy)**: single REST API in `Backend/app` (entry: `Backend/app/main.py`).
- **Database**: PostgreSQL, service name `db` in `Backend/docker-compose.yml`. `DATABASE_URL` env var is used.
- **Frontend**: Next.js (in `Frontend/`) — not required for backend changes but share the API contract.
- **Mobile**: Android (Kotlin) and iOS (Swift) clients in `Mobile/`.
- See `CLAUDE.md` for a full high-level diagram and entity relationships.

### Key files to inspect first

- `CLAUDE.md` — project overview and architecture (single source for 'why').
- `Backend/app/main.py` — HTTP routes and dependency wiring (how services are injected).
- `Backend/app/services/course_service.py` — main service layer example (business logic and DB access).
- `Backend/app/models/` — SQLAlchemy models (`base.py`, `course_rating.py`) and the `to_dict()` pattern.
- `Backend/Makefile` and `Backend/docker-compose.yml` — primary developer commands and local environment.
- `Backend/pyproject.toml` — dependencies and dev-dependencies (pytest, httpx, alembic).

### Developer workflows (explicit commands)

- Start local dev environment (uses Docker Compose):
  - `cd Backend` then `make start`  # starts `db` and `api` containers
- Run DB migrations:
  - `cd Backend` then `make migrate`  # runs Alembic inside the `api` container
- Create a new migration:
  - `cd Backend` then `make create-migration`  # prompts for message and runs autogenerate
- Seed data:
  - `cd Backend` then `make seed` or `make seed-fresh`
- Logs:
  - `cd Backend` then `make logs`
- Run tests (dev environment):
  - Prefer running tests in the backend container context: `cd Backend` then `uv run pytest`
  - Fallback local: `python -m pytest Backend/app/tests` (requires local Python env & DB config)
- Run API without Docker (dev machine):
  - `uvicorn app.main:app --reload --port 8000` (run from `Backend/app` or set PYTHONPATH accordingly)

Notes: the `Makefile` uses the `uv` CLI (see Makefile commands). Follow the `uv run ...` pattern when imitating
project scripts to ensure consistent runtime behavior.

### Project conventions & patterns (concrete)

- Service layer: business logic lives in `Backend/app/services/*.py`. Endpoints in `main.py` should be thin and
  delegate to services (see `get_course_service` dependency).
- Soft-delete semantics: models inherit `deleted_at` from `app/models/base.py`. Queries almost always filter
  `Model.deleted_at.is_(None)` — preserve that when adding queries or migrations.
- Model → response pattern: models often implement `to_dict()` (e.g. `CourseRating.to_dict()`) and services
  return serializable dicts that Pydantic schemas consume (look at `app/schemas/rating.py`).
- Rating rules (domain-specific): rating value must be 1–5; exactly one active rating per `(course_id, user_id)`;
  updates vs creation semantics are implemented in `CourseService.add_course_rating`.
- DB session usage: `get_db` from `Backend/app/db/base.py` provides `Session` objects. Services expect a Session
  (see `get_course_service` in `main.py`). Prefer dependency injection to create services in endpoints.

### Migrations & schema changes checklist

1. Update SQLAlchemy model in `Backend/app/models/`.
2. Run `cd Backend && make create-migration` and provide a descriptive message.
3. Run `cd Backend && make migrate` to apply locally (CI also runs migrations as part of workflow).
4. If seed data required: `make seed` or `make seed-fresh`.

### Tests and CI notes

- Unit and integration tests live under `Backend/app/tests/` and use `pytest` + `httpx`.
- Many tests expect database migrations/seeds; run the Docker-based environment for reliable integration runs.
- The repo's GitHub workflows reference `CLAUDE.md` for review guidance — keep that doc up to date.

### When you edit code as an AI agent

- Keep changes small and focused. Follow existing patterns in `CourseService` and `main.py`.
- Respect `deleted_at` semantics and `to_dict()` usage for API responses.
- Always run/create migrations for model changes and include them in the same PR.
- Prefer adding tests under `Backend/app/tests/` that exercise the service layer (faster and deterministic).

### Where to get more context

- `CLAUDE.md` — system-level overview, entity diagrams, API contracts.
- `Backend/README.md` — backend-specific summary and quick facts.
- Review `Backend/app/services/course_service.py` to learn standard query/aggregation approaches.

If any section is unclear or you want the file to emphasize other parts (e.g., frontend dev flow or mobile), tell me
which area to expand and I'll iterate.
