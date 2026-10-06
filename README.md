# Business Selection Engine

A V0 decision engine for ranking online business opportunities before committing serious build effort.

## Core principle

> Selection quality > effort.

The engine is designed to cheaply scan many opportunities, aggressively eliminate weak ones, spend deeper research budget only on finalists, and concentrate resources on the best validated niche.

## V0 scope

V0 implements:

- structured opportunity records
- evidence tracking
- information gaps
- deterministic scoring
- confidence separated from score
- ranking and selection gates
- research-budget allocation
- prioritized research questions
- REST API
- CLI
- seed dataset
- tests

It deliberately does **not** implement autonomous sales, ad spend, account creation, scraping of prohibited sources, or other commercial actions.

See `CODEX_TASK.md` for the full implementation spec.

## Suggested stack

- Python 3.12+
- FastAPI
- Pydantic
- SQLAlchemy
- PostgreSQL
- Alembic
- pytest
- Docker Compose

## Main development objective

Prove that the system can distinguish strong opportunities from mediocre ones before adding agent complexity.
