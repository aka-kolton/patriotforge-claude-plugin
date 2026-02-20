---
description: "Use after finishing implementation to commit, run a multi-agent review swarm (security, lint, bandit, trivy, pip-audit, tests, code review), auto-fix all new issues, and merge when clean."
---

# PatriotForge Ship Pipeline

Automated commit → review swarm → fix loop → merge. Invoke after implementation is complete.

`$ARGUMENTS` — optional commit message or PR title

---

## Phase 1: Commit & Push

1. Run `git status`, `git diff --stat`, and `git log --oneline -5` in `D:/PatriotForge`
2. Identify base branch (should be `main`)
3. If currently ON `main`, **stop** — tell the user to create a feature branch first (`git checkout -b feature/<name>`)
4. Stage all changed files — EXCLUDE `.env`, `*.key`, credentials, secrets, `__pycache__/`, `node_modules/`
5. Create a commit:
   - Use `$ARGUMENTS` as the message if provided
   - Otherwise generate one following PatriotForge conventions: `feat:`, `fix:`, `chore:`, or `refactor:` prefix
   - End with `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
6. Push with `-u` to set upstream tracking

## Phase 2: Create or Find PR

1. Check for existing PR: `gh pr list --head "$(git branch --show-current)" --json number,url --jq '.[0]'`
2. If none exists, create one:
   ```
   gh pr create --title "<title>" --body "## Summary\n- ...\n\n## Test plan\n- [ ] ..."
   ```
3. Store the PR number and URL
4. Build the **PR file list**: `git diff main...HEAD --name-only`
5. Save the **PR diff** to a temp file: `git diff main...HEAD > /tmp/patriotforge-pr-diff.txt`

## Phase 3: Review Swarm

Spawn ALL FIVE review agents **in parallel** using the `Task` tool. Each agent receives the list of changed files and the diff path. Each must output issues in this exact format (or `NO ISSUES FOUND`):

```
ISSUE | <file>:<line> | <CRITICAL|HIGH|MEDIUM|LOW> | <description> | <suggested fix>
```

### Agent 1: Lint, Format & Types (`subagent_type: "general-purpose"`)

```
Run PatriotForge lint/format/type checks on ONLY the files changed in this PR.
This agent mirrors the CI pipeline checks that run on push to main.

Python files (.py) — run from D:/PatriotForge/backend/:
  python -m ruff check <file>
  python -m ruff format --check <file>
  D:/PatriotForge/backend/.venv/Scripts/python.exe -m mypy --strict <file>

TypeScript/React files (.ts/.tsx) — run from D:/PatriotForge/frontend/:
  cmd /c "cd /d D:\PatriotForge\frontend & npx tsc --noEmit"
  cmd /c "cd /d D:\PatriotForge\frontend & npx eslint <file>"

Note: mypy may report errors on unchanged files due to import chains — only report
errors where the file itself is in the changed files list.

Output format per issue:
ISSUE | <file>:<line> | <severity> | <description> | <suggested fix>

If no issues: NO ISSUES FOUND

Changed files: <PR file list>
```

### Agent 2: Security Review (`subagent_type: "patriotforge:security-reviewer"`)

```
Review ALL changed files for security vulnerabilities introduced by this PR.
Read the diff at D:/PatriotForge to see exactly which lines are new.

PatriotForge-specific checks:
- Parameterized queries only — no SQL string building
- Pydantic extra='forbid' on all request schemas
- CSRF token required on state-changing endpoints
- No localStorage for secrets — HTTP-only cookies only
- Stripe: Checkout Sessions only, webhook sig verified
- Input validation: max lengths, magic bytes on uploads
- No plaintext passwords or secrets in code/logs
- Webhook signatures verified before processing

Also check OWASP top 10: injection, XSS, auth bypass, SSRF, path traversal, insecure deserialization.

ONLY flag issues on lines ADDED or MODIFIED in this PR — ignore pre-existing patterns.

Output format per issue:
ISSUE | <file>:<line> | <CRITICAL|HIGH|MEDIUM|LOW> | <description> | <suggested fix>

If no issues: NO ISSUES FOUND

Changed files: <PR file list>
```

### Agent 3: Test Analysis (`subagent_type: "patriotforge:tdd-agent"`)

```
Analyze test coverage for code changed in this PR. Do NOT write tests — only report gaps.

1. Read the diff to identify new functions, endpoints, services, or logic branches
2. Check for corresponding test files in D:/PatriotForge/backend/tests/
3. Run the test suite:
   SESSION_SECRET=test-secret-that-is-long-enough ENCRYPTION_KEY=dGVzdC1lbmNyeXB0aW9uLWtleS0zMmJ5dGVzISE= DATABASE_URL="sqlite+aiosqlite:///:memory:" REDIS_URL="redis://fake" D:/PatriotForge/backend/.venv/Scripts/python.exe -m pytest tests/ --tb=short -q
4. Report: untested new code paths, failing tests, missing test files

PatriotForge test conventions:
- Tests should cover: success, 422, 401, 429, 409 cases
- No mocking internal layers — test through real service
- aiosqlite + fakeredis for isolation

Output format per issue:
ISSUE | <file>:<line> | <HIGH|MEDIUM> | <description> | <suggested fix>

If no issues: NO ISSUES FOUND

Changed files: <PR file list>
```

### Agent 4: Code Review (`subagent_type: "patriotforge:code-reviewer"`)

```
Review all changed files for logic bugs, convention violations, and quality issues.

Read CLAUDE.md at D:\PatriotForge\CLAUDE.md for full conventions. Key rules:
- All functions async — no sync DB or Redis calls
- Service layer has no HTTP objects (no Request, Response, HTTPException)
- Domain exceptions in services, HTTP mapping in routers
- response_model and status_code on every endpoint
- Dependency injection via Depends() — no global state
- Pydantic v2: model_dump() not dict(), from_attributes not orm_mode
- forge_ table prefix, UUID PKs, NUMERIC(12,2) for money
- Soft delete (deleted_at) — never hard delete
- TypeScript strict — no any without justification
- Tailwind utilities only — no custom CSS

ONLY flag issues on lines ADDED or MODIFIED. Only report HIGH confidence issues.

Output format per issue:
ISSUE | <file>:<line> | <HIGH|MEDIUM> | <description> | <suggested fix>

If no issues: NO ISSUES FOUND

Changed files: <PR file list>
```

### Agent 5: Dependency & OWASP Scanning (`subagent_type: "general-purpose"`)

```
Run security scanning tools that mirror the PatriotForge CI pipeline.
These catch vulnerable dependencies and common security anti-patterns.
Run ALL of these from D:/PatriotForge/:

1. Bandit (Python security linter) — run on changed .py files only:
   D:/PatriotForge/backend/.venv/Scripts/bandit.exe -r <file> -f json
   Bandit flags: hardcoded passwords, eval/exec, subprocess shells, weak crypto, etc.
   Map bandit severity: HIGH->HIGH, MEDIUM->MEDIUM, LOW->LOW
   IMPORTANT: Only report findings in files changed by this PR.

2. pip-audit (Python dependency vulnerabilities):
   D:/PatriotForge/backend/.venv/Scripts/pip-audit.exe --requirement D:/PatriotForge/backend/requirements.txt --format json 2>&1 || true
   If pip-audit finds vulnerabilities, only report HIGH+ severity.
   If a requirements file doesn't exist, try: cd D:/PatriotForge/backend && .venv/Scripts/pip-audit.exe

3. npm audit (JS/TS dependency vulnerabilities):
   cmd /c "cd /d D:\PatriotForge\frontend & npm audit --json"
   Only report HIGH and CRITICAL severity vulnerabilities.

4. Trivy (filesystem vulnerability scan):
   trivy fs D:/PatriotForge --severity HIGH,CRITICAL --format json 2>&1 || C:/Users/Kolton/AppData/Local/Microsoft/WinGet/Packages/AquaSecurity.Trivy_Microsoft.Winget.Source_8wekyb3d8bbwe/trivy.exe fs D:/PatriotForge --severity HIGH,CRITICAL --format json 2>&1
   If neither trivy command works, output: INFO | trivy | LOW | Trivy not found — reinstall with winget install AquaSecurity.Trivy | Reinstall trivy

5. Gitleaks (secrets detection) — scan for leaked secrets in the PR diff:
   gitleaks detect --source D:/PatriotForge --no-banner --report-format json 2>&1 || C:/Users/Kolton/AppData/Local/Microsoft/WinGet/Packages/Gitleaks.Gitleaks_Microsoft.Winget.Source_8wekyb3d8bbwe/gitleaks.exe detect --source D:/PatriotForge --no-banner --report-format json 2>&1
   Gitleaks flags: API keys, tokens, passwords, private keys, connection strings in code.
   IMPORTANT: Only report findings in files changed by this PR.
   If neither gitleaks command works, output: INFO | gitleaks | LOW | Gitleaks not found — reinstall with winget install Gitleaks.Gitleaks | Reinstall gitleaks

For bandit findings: output one ISSUE line per finding in changed files.
For dependency audit findings: output one ISSUE line per vulnerable package.
For gitleaks findings: output one ISSUE line per leaked secret — ALWAYS CRITICAL severity.

Output format per issue:
ISSUE | <file-or-package>:<line-or-version> | <CRITICAL|HIGH|MEDIUM|LOW> | <description> | <suggested fix>

If no issues: NO ISSUES FOUND
```

## Phase 4: Triage

After all five agents return:

1. Parse all `ISSUE | ...` lines from agent outputs
2. Verify each issue is on a line actually changed in this PR:
   - Run `git diff main...HEAD -- <file>` for each flagged file
   - If the flagged line is NOT in the added/modified range, discard it (pre-existing)
3. Deduplicate (same file:line from multiple agents → keep highest severity)
4. Sort: CRITICAL > HIGH > MEDIUM > LOW
5. Print summary:
   ```
   Review complete: X issues found (Y pre-existing filtered out)
   CRITICAL: N | HIGH: N | MEDIUM: N | LOW: N
   ```
6. List each remaining issue

If **0 issues** → skip to Phase 6.

## Phase 5: Fix Loop

**Maximum 3 iterations.**

Each iteration:

1. Fix all identified issues:
   - **Lint/format issues**: run `python -m ruff format <file>` and `cmd /c "cd /d D:\PatriotForge\frontend & npx eslint --fix <file>"`
   - **mypy type errors**: edit code to fix type annotations
   - **Bandit findings**: edit code to fix security anti-patterns (e.g., replace `subprocess.call(shell=True)` with `subprocess.run(...)`)
   - **Security/logic/convention issues**: edit files directly with the Edit tool
   - **Dependency vulnerabilities (pip-audit/npm audit)**: these CANNOT be auto-fixed silently — report them to the user with the suggested version bump and ask whether to update the dependency
   - **Trivy findings**: report to user, cannot auto-fix
2. After all fixes, run formatters:
   - `python -m ruff format` on all changed .py files
   - `cmd /c "cd /d D:\PatriotForge\frontend & npx eslint --fix <files>"` on changed .ts/.tsx
3. Stage and commit:
   ```
   fix: address review findings (round N)

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   ```
4. Push to branch
5. **Re-run Phase 3** (full review swarm)
6. **Re-run Phase 4** (triage)
7. If 0 issues → Phase 6
8. If iteration 3 with remaining issues → stop, report to user, do NOT merge

## Phase 6: Merge

1. Summarize: fix iterations completed, final state, PR URL
2. Ask with `AskUserQuestion`:
   - "All reviews pass. How should I merge?"
   - Options: "Squash merge (Recommended)", "Merge commit", "Don't merge yet"
3. If approved: `gh pr merge <number> --squash --delete-branch` (or `--merge`)
4. If declined: leave PR open, report the URL

## Ground Rules

- **Never fix pre-existing issues** — the triage filter is the most critical step
- **Never auto-merge** — always ask first
- **Max 3 fix cycles** — prevents infinite loops
- **All commits get co-author tag**
- **Never use `sed` or `awk`** — use Edit tool or Python scripts
- **Always use forward slashes** in Bash paths (`D:/PatriotForge/...`)
- If `gh` CLI is missing or git is broken, stop and tell the user
