# reviewing-security

Pre-release security audit — nine dedicated axes with ship-gate decisions.

## When to Use

Say any of the following to activate:
- "security review"
- "security audit"
- "review for security"
- "is this secure"
- "before we ship"

Also activates automatically when code touches: auth flows, permission checks, PII fields, payment logic, file upload/download.

## What It Does

Audits code across nine axes. Every axis is reported — "No issues found" is explicitly stated for clean axes.

| Axis | Focus |
|------|-------|
| 1 — Injection | SQL, Command, LDAP, XSS |
| 2 — Authentication | Hashing, sessions, JWT |
| 3 — Authorization | Horizontal & vertical privilege escalation |
| 4 — Sensitive Data | PII logging, over-fetching, HTTPS |
| 5 — Misconfiguration | CORS, security headers, debug mode |
| 6 — CSRF/SSRF | Token validation, URL allowlists |
| 7 — Secrets | Hardcoded credentials, .gitignore |
| 8 — Input Validation | Boundary checks, file uploads |
| 9 — Dependencies | Known CVEs |

## Severity Scale & Ship Gate

| Severity | Ship Gate |
|----------|-----------|
| CRITICAL | Block release |
| HIGH | Fix in current sprint |
| MEDIUM | Fix in next version |
| LOW | Track as tech debt |

## vs. auto-code-review

`auto-code-review` runs a single security pass on every code generation.
`reviewing-security` runs a full nine-axis audit before production deploys.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill instructions |
| `docs/SECURITY_CHECKLIST.md` | Printable pre-release checklist for engineers |
