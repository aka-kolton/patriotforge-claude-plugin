# Changelog

## 1.3.0 (2026-02-19)

### Added
- **debug** skill — production triage pipeline: dispatches 4-6 scouts in parallel to check server logs, container health, database state, recent deploys, network connectivity, and codebase for root cause analysis
- **qa** skill — live browser testing with Playwright MCP: walks through 8 user flows (login, dashboard, customers, quotes, orders, production, navigation, settings), catches console errors, network failures, broken UI, and logs bugs with steps to reproduce
- **gitleaks** v8.30.0 added to ship skill's Agent 5 for secrets detection (completes full CI parity)

### Changed
- **patriotdevbot** updated: added scout + planner to agent roster, replaced manual 5-step validate/review loop with /ship pipeline, uses scout instead of Explore for PatriotForge-aware lookups

## 1.2.0 (2026-02-19)

### Added
- **scout** agent (haiku) — fast, cheap investigative agent for codebase recon, server SSH, Railway status, and web research. Designed to run 3-5 in parallel for broad coverage.
- **planner** agent (sonnet) — deep analysis and architecture planning agent. Synthesizes scout findings into phased implementation plans following PatriotForge conventions.
- **plan** skill — investigative planning pipeline: dispatches scouts (haiku) for fast recon, then planners (sonnet) for architecture. Covers local codebase, server, Railway, and external API research. Saves plans to Obsidian logbook.

## 1.1.0 (2026-02-19)

### Added
- **ship** skill — automated shipping pipeline: commit, 5-agent parallel review swarm, auto-fix loop, merge
  - Agent 1: Lint, Format & Types (ruff, eslint, tsc, mypy --strict)
  - Agent 2: Security Review (OWASP top 10, PatriotForge security rules)
  - Agent 3: Test Analysis (pytest suite, coverage gap detection)
  - Agent 4: Code Review (logic bugs, convention violations)
  - Agent 5: Dependency & OWASP Scanning (bandit, pip-audit, npm audit, Trivy)
- Pre-existing issue filter — only flags/fixes issues on lines changed in the PR
- Auto-fix loop with 3-iteration cap to prevent infinite cycles
- User confirmation required before merge

## 1.0.0 (2026-02-13)

### Added
- 12 specialized agents for full-stack development orchestration
  - **patriotdevbot**: autonomous orchestrator (opus)
  - **backend-dev**: FastAPI/SQLAlchemy implementation (sonnet)
  - **database-dev**: database models and Alembic migrations (sonnet)
  - **api-dev**: Pydantic schemas and API contracts (sonnet)
  - **tdd-agent**: pytest test writing and fixtures (sonnet)
  - **frontend-dev**: React/TypeScript implementation (sonnet)
  - **ui-agent**: TailwindCSS styling and component design (sonnet)
  - **ux-agent**: UX analysis and accessibility (sonnet)
  - **security-reviewer**: security audits (opus, read-only)
  - **code-reviewer**: convention compliance review (opus, read-only)
  - **railway-agent**: Docker/Railway deployment (sonnet)
  - **github-agent**: git workflow and PR management (sonnet)
- 8 domain skills migrated from PatriotForge project
  - backend, frontend, database, api, security, tdd, code-review, deploy
- Plugin manifest with full metadata
- MIT license
