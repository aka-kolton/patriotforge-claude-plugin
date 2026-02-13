---
description: "Use when writing tests, setting up test fixtures, mocking dependencies, or following test-driven development — pytest-asyncio, fakeredis, httpx, and test organization patterns."
---

# PatriotForge TDD Patterns

**Philosophy:** Tests first, implementation follows. Every service function and endpoint gets tests before code.

**Stack:** pytest-asyncio (auto mode) · fakeredis · aiosqlite · httpx AsyncClient

## Fixture Hierarchy

```python
# conftest.py — shared fixtures cascade top-down:
test_settings()      # → Settings with SQLite + fakeredis + short timeouts
test_engine()        # → async SQLAlchemy engine (in-memory SQLite)
test_db_session()    # → async session factory, creates tables per-test
test_redis()         # → FakeRedis (async), flushed per-test
app(settings, session_factory, redis)  # → FastAPI app with test state injected
client(app)          # → httpx.AsyncClient with ASGI transport
```

**Key:** The `app` fixture injects test dependencies via `app.state`, so no monkeypatching is needed.

## Test File Organization
```
backend/tests/
├── conftest.py                 # shared fixtures + helpers
├── test_auth_service.py        # unit tests — service layer
├── test_auth_api.py            # integration tests — full HTTP
├── test_quotes_service.py
├── test_quotes_api.py
└── ...
```
- `test_<module>_service.py` — tests service functions directly (no HTTP)
- `test_<module>_api.py` — tests endpoints via `client` (full request/response)

## Class-Based Grouping
```python
class TestPasswordHashing:
    """Tests for hash_password() and verify_password()."""

    async def test_hashes_are_salted(self, test_settings):
        ...

    async def test_rejects_short_passwords(self, test_settings):
        ...
```
- Group by function or feature
- Docstrings on classes, descriptive method names

## Helper Functions
```python
# In conftest.py — reusable across test files:
async def register_user(client, **overrides) -> httpx.Response:
    payload = {"email": "test@example.com", "password": "SecurePass123!", ...}
    payload.update(overrides)
    return await client.post("/api/auth/register", json=payload)

async def login_user(client, email, password) -> httpx.Response:
    return await client.post("/api/auth/login", json={...})
```

## Coverage Checklist (Per Endpoint)
- ✅ Success case (correct status code + response body)
- ✅ 422 — validation errors (missing fields, bad types, extra fields)
- ✅ 401 — unauthenticated access
- ✅ 429 — rate limit exceeded
- ✅ 409 — conflict (duplicate resources)
- ✅ Cookie attributes (HttpOnly, Secure, SameSite, Path, Max-Age)
- ✅ Security headers (Cache-Control: no-store, no password_hash leaks)
- ✅ Edge cases (inactive user, expired session, concurrent requests)

## Testing Anti-Patterns to Avoid
- No mocking of internal service functions — test through the real layer
- No sleeping for timing — use short test timeouts (2s idle, 5s absolute)
- No shared state between tests — each test gets a fresh DB + Redis

📖 **Reference:** `backend/tests/conftest.py`, `backend/tests/test_auth_service.py`, `backend/tests/test_auth_api.py`
