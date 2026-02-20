---
description: "Use to live-test the PatriotForge app with Playwright MCP. Walks through user flows, catches console errors, network failures, broken UI, and logs bugs found with steps to reproduce."
argument-hint: Optional — specific feature or flow to test, or "full" for all flows
---

# PatriotForge QA — Live Testing with Playwright

Automated browser testing of the live PatriotForge app. Uses Playwright MCP to navigate, interact, and catch bugs.

`$ARGUMENTS` — specific flow to test (e.g., "quotes", "login", "dashboard") or "full" for all flows

---

## Setup

1. Navigate to the app:
   - **Production:** `https://forge.patriotpf.com` (or `https://frontend-production-f8a3.up.railway.app`)
   - **Local dev:** `http://localhost:5173` (if running locally)
   - Ask with `AskUserQuestion` if neither is specified: "Which environment should I test?"

2. Use `mcp__playwright__browser_navigate` to open the app
3. Take initial screenshot to confirm the app loads

---

## Test Flows

Run the flows relevant to `$ARGUMENTS`, or all flows if "full" is specified. For each flow:

1. **Take a screenshot** before each major action
2. **Check console** for errors after each page navigation: `mcp__playwright__browser_console_messages` with level "error"
3. **Check network** for failed requests: `mcp__playwright__browser_network_requests` — look for 4xx/5xx responses
4. **Take a screenshot** on any failure for evidence

### Flow 1: Login
```
1. Navigate to login page
2. Verify login form renders (username/email field, password field, submit button)
3. Try login with test credentials (ask user for credentials if needed)
4. Verify redirect to dashboard after login
5. Check: auth cookie set, no console errors, no network failures
```

### Flow 2: Dashboard
```
1. After login, verify dashboard loads
2. Check all dashboard widgets/cards render
3. Verify sidebar navigation is visible
4. Click through each top-level nav item — verify each page loads
5. Check: no blank pages, no loading spinners stuck, no console errors
```

### Flow 3: Customer Management
```
1. Navigate to Customers page
2. Verify customer list renders with data (or empty state)
3. Click "New Customer" — verify form renders
4. Fill required fields, submit (or cancel)
5. If submitted: verify customer appears in list
6. Click a customer row — verify detail page renders
7. Check: form validation works, no console errors
```

### Flow 4: Quotes
```
1. Navigate to Quotes page
2. Verify quote list renders
3. Click "New Quote" — verify form renders
4. Check: customer selector works, line items can be added
5. Verify price calculations display correctly
6. Test status transitions if possible (Draft → Sent)
7. Check: no console errors, no network failures
```

### Flow 5: Sales Orders
```
1. Navigate to Sales Orders page
2. Verify list renders with correct columns
3. Click into a sales order — verify detail page
4. Check: line items display, pricing correct, status badge renders
5. Check: no console errors
```

### Flow 6: Work Orders & Production
```
1. Navigate to Work Orders or Production page
2. Verify list renders
3. Check status badges display correct colors
4. Click into a work order — verify detail view
5. Check: status flow matches expected (Queued → In Production → QC → Complete)
```

### Flow 7: Navigation & Permissions
```
1. Click through EVERY sidebar nav item
2. For each page: verify it loads (no blank screen, no crash)
3. Check for permission-gated content (if logged in as non-admin)
4. Verify responsive behavior: resize browser to tablet (768px) and mobile (375px)
5. Check: sidebar collapses correctly, no content overflow
```

### Flow 8: Settings & Admin
```
1. Navigate to Settings page
2. Verify settings categories render
3. Check user management page (if admin)
4. Verify role/permission display
```

---

## Bug Detection Checks

After EVERY page navigation or action, run these checks:

### Console Errors
```
mcp__playwright__browser_console_messages with level "error"
```
Flag any errors that aren't known/expected (e.g., React strict mode double-render warnings are OK).

### Network Failures
```
mcp__playwright__browser_network_requests
```
Flag any requests with status 4xx or 5xx. Record the URL, method, and status code.

### Visual Checks
Take screenshots and verify:
- No blank/white pages
- No stuck loading spinners (wait up to 5 seconds)
- No overlapping elements or broken layouts
- Status badges show correct colors
- Tables have data or proper empty states
- Forms have labels and proper field types

### Accessibility
While navigating, note:
- Missing alt text on images
- Buttons/links without text or aria-labels
- Color contrast issues (if obvious)
- Focus order issues

---

## Bug Report Format

For each bug found, log it in this format:

```markdown
### BUG: {Short description}

**Severity:** CRITICAL / HIGH / MEDIUM / LOW
**Page:** {URL path}
**Flow:** {Which test flow}

**Steps to Reproduce:**
1. Navigate to {page}
2. Click {element}
3. Observe {problem}

**Expected:** {what should happen}
**Actual:** {what actually happens}

**Evidence:**
- Console error: `{error message}`
- Network failure: `{method} {url} → {status}`
- Screenshot: {reference to screenshot taken}

**Suggested Fix:** {if obvious from the error/code}
```

---

## Phase Summary

After all flows are tested:

1. **Present bug summary:**
   ```
   QA RESULTS — {environment}
   ===========================
   Tested: {N} flows
   Bugs found: {N} (X critical, Y high, Z medium, W low)
   Console errors: {N}
   Network failures: {N}
   Pages tested: {N}
   ```

2. **List all bugs** in severity order

3. **Ask with `AskUserQuestion`:**
   - "Want me to fix these bugs now?"
   - Options: "Fix all", "Fix critical/high only", "Just log them", "Create GitHub issues"

4. If fixing: dispatch appropriate agents for each bug
5. If logging: save bug report to `C:\Users\Kolton\Documents\Logbook\Projects\PatriotForge\Docs\QA Reports\{date}-qa-report.md`

---

## Ground Rules

- **Take screenshots** at every step — evidence matters
- **Check console and network** after every navigation — don't skip
- **Never enter real credentials** — ask user for test account or skip auth-required flows
- **Never modify production data** — use test/draft records only
- **Wait for pages to load** — use `mcp__playwright__browser_wait_for` before asserting
- **Max 3 retries** on flaky loads before flagging as a bug
- **Don't test destructive actions** (delete, void, cancel) unless explicitly asked
- **Respect bot detection** — if CAPTCHA appears, stop and report it
