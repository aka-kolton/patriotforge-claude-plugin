---
description: "Use when something is broken in production or behaving unexpectedly. Dispatches scouts to check server logs, container status, database state, Railway health, and recent deploys in parallel."
argument-hint: What's broken — error message, symptom, or affected feature
---

# PatriotForge Debug & Triage

Rapid parallel investigation when something is broken or behaving unexpectedly.

`$ARGUMENTS` — the symptom, error, or affected feature

---

## Phase 1: Understand the Problem

1. If `$ARGUMENTS` is provided, summarize the issue
2. If vague, ask with `AskUserQuestion`:
   - What's the symptom? (error message, broken page, wrong data, etc.)
   - When did it start? (after a deploy, randomly, after a specific action)
   - Which app? (Portal, Floor Tracker, Label Quoter, Forge)
3. Determine investigation targets based on the symptom

---

## Phase 2: Rapid Triage (Scouts — haiku, all in parallel)

Launch **4-6 scout agents** (`subagent_type: "patriotforge:scout"`) simultaneously. Pick the most relevant:

### Server Scout — Container Health
```
SSH to patriotdev@100.111.104.5 and check:
1. Container status: docker ps | grep patriot (are all containers running?)
2. Container restarts: docker inspect --format='{{.RestartCount}}' <container>
3. Memory/CPU: docker stats --no-stream | grep patriot
4. Disk space: df -h /srv/docker

Report any containers that are down, restarting, or resource-starved.
```

### Log Scout — Recent Errors
```
SSH to patriotdev@100.111.104.5 and check recent logs:
1. cd /srv/docker/patriot-portal
2. docker-compose logs --tail=100 <service> 2>&1 | grep -iE "error|exception|traceback|critical|fatal"
3. Check the last 5 minutes: docker-compose logs --since=5m <service>

Service to check: {affected service based on symptom}
Report: timestamps, error messages, stack traces, frequency.
```

### Database Scout — Data State
```
SSH to patriotdev@100.111.104.5 and check database:
1. Connection count: docker-compose exec -T patriot-db psql -U postgres -d printshop -c "SELECT count(*) FROM pg_stat_activity;"
2. Recent migrations: docker-compose exec -T <backend> alembic current
3. If specific data issue: run targeted query on the affected table
4. Check for locks: docker-compose exec -T patriot-db psql -U postgres -d printshop -c "SELECT * FROM pg_locks WHERE NOT granted;"

Report: migration state, connection issues, data anomalies.
```

### Deploy Scout — Recent Changes
```
SSH to patriotdev@100.111.104.5 and check:
1. Recent git history: cd /srv/docker/patriot-portal && git log --oneline -10
2. Sub-repo history: git -C <subrepo> log --oneline -5
3. Recent image builds: docker images --format '{{.Repository}} {{.CreatedAt}}' | grep patriot | head -10
4. Check if running code matches latest commit

Also check locally:
5. git log --oneline -10 in D:/PatriotForge
6. Any uncommitted changes: git status

Report: what changed recently, any mismatch between local and deployed.
```

### Network Scout — Connectivity
```
SSH to patriotdev@100.111.104.5 and check:
1. Caddy status: docker-compose logs --tail=20 patriot-caddy
2. Can backend reach database: docker-compose exec -T <backend> python -c "import asyncio; print('db ok')" 2>&1
3. Can backend reach Redis: docker-compose exec -T <backend> python -c "import redis; r=redis.Redis(); r.ping(); print('redis ok')" 2>&1
4. DNS/SSL issues in Caddy logs

Report: any connectivity failures between services.
```

### Codebase Scout — Local Code Check
```
Search the PatriotForge codebase at D:/PatriotForge for code related to {symptom}.
Check for:
1. Recent changes to affected files (git diff HEAD~5 -- <relevant paths>)
2. The specific endpoint, component, or service involved
3. Any TODO/FIXME/HACK comments in the area
4. Error handling (or lack thereof) in the affected code path

Report: relevant code paths, recent changes, potential root causes.
```

---

## Phase 3: Diagnosis

After all scouts return:

1. **Correlate findings** across scouts:
   - Did a recent deploy coincide with the issue?
   - Are there resource constraints causing failures?
   - Is it a code bug, config issue, or infrastructure problem?
2. **Identify root cause** (or top 2-3 candidates if uncertain)
3. **Present findings** to the user:
   ```
   DIAGNOSIS
   ---------
   Symptom: {what's broken}
   Root cause: {what's causing it}
   Evidence: {log entries, data, deploy timing}

   Affected: {services, endpoints, components}
   Since: {when it started, correlated event}
   Impact: {CRITICAL / HIGH / MEDIUM / LOW}
   ```

---

## Phase 4: Recommended Fix

Based on diagnosis, recommend the fix:

### If it's a code bug:
- Identify the exact file:line and the fix needed
- Offer to fix it now (edit code directly)

### If it's a config/infra issue:
- Provide the exact SSH commands to resolve it
- Ask user for confirmation before running destructive commands

### If it's a deploy issue:
- Recommend rollback steps or re-deploy procedure
- Reference the rebuild commands from CLAUDE.md

### If it's unclear:
- List what was ruled out
- Suggest next investigation steps
- Ask the user for more context

---

## Ground Rules

- **Speed over thoroughness** — get to the root cause fast
- **All scouts run in parallel** — don't wait for one before starting another
- **Never fix without asking** — present the diagnosis, then ask before applying fixes
- **Never run destructive commands** on the server without explicit approval
- **Use forward slashes** in Bash paths (`D:/PatriotForge/...`)
- **SSH target:** `patriotdev@100.111.104.5`
- **Docker compose dir:** `/srv/docker/patriot-portal/`
