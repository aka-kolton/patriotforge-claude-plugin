---
description: "Use when writing Python backend code — FastAPI routers, SQLAlchemy models, Pydantic schemas, service functions, dependency injection, or app configuration."
---

# PatriotForge Backend Patterns

**Stack:** FastAPI · Python 3.12 · SQLAlchemy 2.x (async) · Pydantic v2 · Redis 7 · PostgreSQL 15

## Project Layout

```
backend/
├── app/
│   ├── main.py          # create_app() factory + lifespan
│   ├── config.py         # Settings(BaseSettings) — env-driven
│   ├── routers/          # one file per module (auth.py, quotes.py, …)
│   ├── services/         # pure business logic — no HTTP objects
│   ├── schemas/          # Pydantic request/response models
│   ├── models/           # SQLAlchemy ORM models
│   └── deps.py           # dependency injection functions
├── tests/                # pytest-asyncio tests
├── alembic/              # database migrations
└── pyproject.toml
```

## Router Pattern
```python
router = APIRouter(prefix="/api/auth", tags=["auth"])

@router.post("/login", response_model=LoginResponse, status_code=200)
async def login(
    data: LoginRequest,
    db: AsyncSession = Depends(get_db_session),
    redis: Redis = Depends(get_redis),
    request: Request, response: Response,
): ...
```
- Always declare `response_model`, `status_code`, explicit `Depends()`
- Router calls service layer — no business logic in routers

## Service Layer
- Pure async functions — accept db session, redis, domain objects
- Raise domain exceptions (`DuplicateEmail`, `InvalidCredentials`), not `HTTPException`
- Routers catch domain exceptions and map to HTTP status codes

## Pydantic v2
- **Requests:** `model_config = ConfigDict(extra='forbid')` — reject unknown fields
- **Responses:** `model_config = ConfigDict(from_attributes=True)` — ORM compatibility
- Validate string lengths, use `SecretStr` for passwords

## SQLAlchemy Models
- Use `Mapped[]` type annotations + `mapped_column()`
- Table names: `forge_` prefix (e.g., `forge_users`)
- UUID primary keys: `mapped_column(UUID, primary_key=True, default=uuid4)`
- Always include: `created_at`, `updated_at`, `deleted_at`

## Dependency Injection
- `get_db_session()`: yields `AsyncSession` from `app.state.async_session_factory`
- `get_redis()`: returns `app.state.redis`
- State-based injection makes testing trivial (swap state in test app)

## App Factory
- `create_app(settings)` builds the app — test-friendly, no module-level singletons
- Lifespan context manager initializes DB engine, Redis, runs Alembic check

## Conventions
- All I/O is async — no sync DB calls
- Strict mypy, ruff formatting, 100-char line limit
- Never use `sed`/`awk` for file editing

📖 **Reference files:** `backend/app/routers/auth.py`, `backend/app/services/auth.py`, `backend/app/schemas/auth.py`
