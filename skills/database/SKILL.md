---
description: "Use when creating database models, writing migrations, designing schemas, or working with PostgreSQL queries — table naming, data types, constraints, and Alembic workflows."
---

# PatriotForge Database Conventions

**Stack:** PostgreSQL 15 · SQLAlchemy 2.x (async) · Alembic · Railway hosting

## Naming & Structure
- All tables use `forge_` prefix: `forge_users`, `forge_quotes`, `forge_line_items`
- Database: shared `printshop` database on Railway PostgreSQL
- Schema: `public` (default)

## Primary Keys
- UUID v4 for all tables — never auto-increment integers
- `mapped_column(UUID, primary_key=True, default=uuid4)`

## Required Columns (Every Table)
| Column | Type | Notes |
|--------|------|-------|
| `id` | `UUID` | PK, default `uuid4()` |
| `created_at` | `DateTime(timezone=True)` | `server_default=func.now()` |
| `updated_at` | `DateTime(timezone=True)` | `onupdate=func.now()` |
| `deleted_at` | `DateTime(timezone=True)` | Nullable — soft delete flag |

## Money & Numeric Types
- **Money:** `NUMERIC(12,2)` — NEVER use `FLOAT` or `DECIMAL` without precision
- **Percentages:** `NUMERIC(5,4)` — stores as 0.0750 for 7.5%
- **Quantities:** `INTEGER` or `NUMERIC(10,2)` for fractional

## Soft Delete
- Filter all queries: `WHERE deleted_at IS NULL`
- Scoped queries: always include `company_id` filter
- Never hard delete — set `deleted_at = now()`

## Document Numbering
- Unified base number with type suffixes:
  - `12345Q` — Quote
  - `12345SO` — Sales Order
  - `12345-01` — Work Order (dash + sequence)
  - `INV12345` — Invoice
  - `PY#` — Payment

## SQLAlchemy Model Pattern
```python
from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase
from sqlalchemy import String, DateTime, func
from uuid import UUID, uuid4

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "forge_users"
    id: Mapped[UUID] = mapped_column(UUID, primary_key=True, default=uuid4)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    deleted_at: Mapped[datetime | None] = mapped_column(DateTime(timezone=True))
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), server_default=func.now())
    updated_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), onupdate=func.now())
```

## Alembic Conventions
- Auto-generate migrations: `alembic revision --autogenerate -m "description"`
- Import ALL models in `alembic/env.py` so autogenerate detects them
- Run on deploy: `alembic upgrade head`
- Two database roles:
  - `forge_app` — runtime (SELECT, INSERT, UPDATE on data tables)
  - `forge_migrate` — migration only (CREATE, ALTER, DROP)

## Constraints & Indexes
- Foreign keys on all relationships
- `CHECK` constraints on enum-like columns
- Unique indexes on natural keys (email, document numbers)
- Composite indexes for common query patterns

📖 **Full schema:** `docs/plans/2026-01-24-database-schema.md`, **Model example:** `backend/app/models/user.py`
