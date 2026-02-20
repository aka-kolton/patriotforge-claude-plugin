# PatriotForge Claude Code Plugin

Full-stack development orchestration for [PatriotForge ERP](https://github.com/aka-kolton/patriotforge) — 14 specialized agents and 12 domain skills for Claude Code.

## What's Included

### 14 Specialized Agents

| Agent | Model | Role |
|-------|-------|------|
| **patriotdevbot** | opus | Orchestrator — reads plans, breaks into phases, dispatches agents, manages PRs |
| **backend-dev** | sonnet | FastAPI routers, SQLAlchemy models, Pydantic schemas, service functions |
| **database-dev** | sonnet | SQLAlchemy models, Alembic migrations, schema design |
| **api-dev** | sonnet | Pydantic schemas, endpoint contracts, middleware configuration |
| **tdd-agent** | sonnet | pytest-asyncio tests, fixtures, coverage verification |
| **frontend-dev** | sonnet | React 19 components, TypeScript, TanStack Query, routing |
| **ui-agent** | sonnet | TailwindCSS styling, component design, responsive layouts |
| **ux-agent** | sonnet | User flows, accessibility, interaction patterns |
| **security-reviewer** | opus | Security audits, auth review, vulnerability detection (read-only) |
| **code-reviewer** | opus | Convention compliance, quality review, silent failure detection (read-only) |
| **railway-agent** | sonnet | Docker, Railway deployment, CI/CD, database migrations |
| **github-agent** | sonnet | Git workflow, PR creation, branch management, CI monitoring |
| **scout** | haiku | Fast codebase/server/Railway recon — run 3-5 in parallel for broad coverage |
| **planner** | sonnet | Deep architecture analysis, phased implementation plans |

### 12 Domain Skills

| Skill | When to Use |
|-------|-------------|
| **backend** | Writing FastAPI routers, services, config |
| **frontend** | Writing React components, pages, styling |
| **database** | Designing tables, writing migrations |
| **api** | Designing endpoints, Pydantic schemas |
| **security** | Reviewing auth, payments, webhooks |
| **tdd** | Writing tests, fixtures, coverage |
| **code-review** | Reviewing PRs, auditing code quality |
| **deploy** | Docker, CI/CD, Railway deployment |
| **ship** | Commit, 5-agent review swarm (ruff, eslint, tsc, mypy, bandit, pip-audit, npm audit, trivy, gitleaks), auto-fix loop, merge |
| **plan** | Investigate and plan features — scouts (haiku) for recon, planners (sonnet) for architecture |
| **debug** | Production triage — parallel scouts check logs, containers, DB, deploys, network |
| **qa** | Live browser testing with Playwright MCP — walks user flows, catches console errors, network failures, logs bugs |

## Workflow

The full development lifecycle:

```
/patriotforge:plan <feature>     →  scouts investigate, planner produces phased plan
patriotdevbot executes plan      →  dispatches agents per phase, commits as it goes
/patriotforge:ship               →  commit, 5-agent review swarm, fix loop, merge
/patriotforge:qa                 →  live browser testing, bug detection
/patriotforge:debug <symptom>    →  parallel triage when something breaks
```

## Installation

### From GitHub (recommended)

```bash
# Step 1: Add the marketplace
/plugin marketplace add aka-kolton/patriotforge-claude-plugin

# Step 2: Install the plugin
/plugin install patriotforge@patriotforge-marketplace
```

### From local directory (development)

```bash
claude --plugin-dir ./plugins/patriotforge
```

## Usage

After installation, agents and skills are namespaced under `patriotforge:`:

```
# Investigate and plan a feature
/patriotforge:plan add Stripe payment processing

# Run the orchestrator on an existing plan
Use the patriotforge:patriotdevbot agent to implement the plan

# Ship when done — commit, review, fix, merge
/patriotforge:ship feat: add payment webhooks

# Live test the deployed app
/patriotforge:qa full

# Debug a production issue
/patriotforge:debug "500 errors on invoice page"
```

## Tech Stack

This plugin is built for the PatriotForge ERP tech stack:

| Layer | Technology |
|-------|-----------|
| Backend | FastAPI (Python 3.12), async SQLAlchemy 2.0, Alembic, Pydantic v2 |
| Frontend | React 19, TypeScript (strict), Vite, TailwindCSS v4, React Router v7 |
| Database | PostgreSQL 15 |
| Cache | Redis 7 |
| Auth | Redis sessions, Argon2 passwords, TOTP MFA |
| Deployment | Railway, Docker, GitHub Actions CI |

## Local Security Tools

The `/ship` skill runs a full CI-equivalent locally:

| Tool | Version | What it checks |
|------|---------|---------------|
| ruff | system | Python lint + format |
| mypy | venv | Python strict type checking |
| eslint | npm | TypeScript/React lint |
| tsc | npm | TypeScript type checking |
| pytest | venv | Python tests |
| bandit | venv | Python security patterns |
| pip-audit | venv | Python dependency CVEs |
| npm audit | npm | JS dependency CVEs |
| trivy | 0.69.1 | Filesystem vulnerability scan (HIGH/CRITICAL) |
| gitleaks | 8.30.0 | Secrets detection in code |

## Project Structure

```
patriotforge-claude-plugin/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace catalog
├── plugins/
│   └── patriotforge/
│       ├── .claude-plugin/
│       │   └── plugin.json    # Plugin manifest
│       ├── agents/            # 14 specialized agents
│       │   ├── patriotdevbot.md
│       │   ├── backend-dev.md
│       │   ├── database-dev.md
│       │   ├── api-dev.md
│       │   ├── tdd-agent.md
│       │   ├── frontend-dev.md
│       │   ├── ui-agent.md
│       │   ├── ux-agent.md
│       │   ├── security-reviewer.md
│       │   ├── code-reviewer.md
│       │   ├── railway-agent.md
│       │   ├── github-agent.md
│       │   ├── scout.md
│       │   └── planner.md
│       └── skills/            # 12 domain skills
│           ├── backend/SKILL.md
│           ├── frontend/SKILL.md
│           ├── database/SKILL.md
│           ├── api/SKILL.md
│           ├── security/SKILL.md
│           ├── tdd/SKILL.md
│           ├── code-review/SKILL.md
│           ├── deploy/SKILL.md
│           ├── ship/SKILL.md
│           ├── plan/SKILL.md
│           ├── debug/SKILL.md
│           └── qa/SKILL.md
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## Customizing for Your Project

The agents reference PatriotForge-specific paths and conventions. To adapt for your own project:

1. Fork this repo
2. Update file paths in agent prompts (search for `D:\PatriotForge`)
3. Update conventions (table prefixes, tech stack, etc.)
4. Update skill content to match your project patterns

## License

MIT
