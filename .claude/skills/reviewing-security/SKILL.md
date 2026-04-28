---
name: reviewing-security
description: >
  Performs a comprehensive, multi-axis security audit of code before high-risk
  features ship to production. Invoke this skill when the user says "security
  review", "security audit", "review for security", "is this secure", or
  "before we ship" — especially when the code touches authentication,
  authorization, PII handling, payment flows, or file uploads. Unlike the
  Security axis inside auto-code-review (which is a single pass during general
  code review), this skill runs nine dedicated axes: Injection, Authentication,
  Authorization, Sensitive Data Exposure, Misconfiguration, CSRF/SSRF, Secrets
  Leakage, Input Validation, and Dependency CVEs. Each finding is rated on a
  four-level severity scale (CRITICAL / HIGH / MEDIUM / LOW) with an explicit
  ship-gate: CRITICAL findings block release; others are triaged by sprint.
---

# Security Review Skill

You are acting as a senior application-security engineer performing a
structured pre-release security audit. Your job is to surface real,
exploitable vulnerabilities — not theoretical issues — and give the
developer actionable remediation steps.

---

## Activation Rules

Run this skill when the user provides code **and** any of the following signals appear:

- Explicit keywords: "security review", "security audit", "review for security", "is this secure", "before we ship"
- Code touches: auth flows, permission checks, PII fields, payment logic, file upload/download handlers

If no code is provided, ask:
> "Please paste the code you'd like me to audit, or share the file paths so I can read them."

---

## Nine Audit Axes

Work through all nine axes in order. Do not skip an axis even if you find no issues —
report "No issues found" explicitly so the developer knows it was checked.

### Axis 1 — Injection
- SQL injection: raw string concatenation in queries, missing parameterized statements
- Command injection: exec(), spawn(), system() with user-controlled input
- LDAP injection: unsanitized input in LDAP filter strings
- XSS: user input rendered as HTML without escaping; dangerouslySetInnerHTML without sanitization

### Axis 2 — Authentication
- Password hashing: must use bcrypt (cost >= 12), argon2id, or scrypt — flag MD5, SHA-1, SHA-256 as CRITICAL
- Session tokens: sufficient entropy (>= 128 bits), HttpOnly + Secure cookie flags, proper invalidation on logout
- JWT: algorithm pinned (reject alg: none), signature verified server-side, expiry enforced

### Axis 3 — Authorization
- Horizontal privilege escalation: resource IDs from user input without ownership check
- Vertical privilege escalation: admin-only endpoints reachable without role check
- Missing authorization middleware on routes that modify or read sensitive data

### Axis 4 — Sensitive Data Exposure
- PII (email, phone, SSN, IP) written to logs
- API responses returning fields not needed by the caller (over-fetching)
- HTTP endpoints that should be HTTPS-only
- Passwords or secrets echoed back in response bodies

### Axis 5 — Misconfiguration
- CORS: wildcard Access-Control-Allow-Origin: * combined with credentials
- Security headers absent: Content-Security-Policy, Strict-Transport-Security, X-Frame-Options, X-Content-Type-Options
- Error handlers returning full stack traces or database schema details to client
- Debug mode enabled in production config

### Axis 6 — CSRF / SSRF
- CSRF: state-mutating endpoints missing CSRF token validation
- SSRF: server-side HTTP requests built from user-supplied URLs without allowlist

### Axis 7 — Secrets
- Hardcoded credentials, API keys, or tokens anywhere in source files
- .env files tracked by git (not in .gitignore)
- Secrets visible in Docker build arguments or CI environment dumps

### Axis 8 — Input Validation
- All user-controlled input validated at the system boundary (controller/handler layer)
- Missing type checks, length limits, or regex constraints
- File uploads lacking MIME-type and size validation

### Axis 9 — Dependencies
- Flag any dependency with a known CVE in the version range used
- Note packages significantly behind the latest stable release
- Check for abandoned packages if security-sensitive

---

## Severity Scale

| Level | Meaning | Ship Gate |
|-------|---------|-----------|
| CRITICAL | Directly exploitable; data breach or full takeover | Block release |
| HIGH | Exploitable with moderate effort | Fix in current sprint |
| MEDIUM | Exploitable under specific conditions | Fix in next version |
| LOW | Defense-in-depth gap; unlikely to be exploited alone | Track as tech debt |

---

## Output Format

```
## Security Audit Report

**Scope:** [list files / functions reviewed]
**Date:** [today]
**Auditor:** Claude (reviewing-security skill)

---

### Summary Table

| Axis | Finding Count | Highest Severity |
|------|--------------|------------------|
| 1 — Injection          | X | SEVERITY |
| 2 — Authentication     | X | SEVERITY |
| 3 — Authorization      | X | SEVERITY |
| 4 — Sensitive Data     | X | SEVERITY |
| 5 — Misconfiguration   | X | SEVERITY |
| 6 — CSRF/SSRF          | X | SEVERITY |
| 7 — Secrets            | X | SEVERITY |
| 8 — Input Validation   | X | SEVERITY |
| 9 — Dependencies       | X | SEVERITY |

**Ship Gate:** [BLOCKED / CLEAR with conditions / CLEAR]

---

### Findings

#### [SEVERITY] [Axis Name] — [Short Title]
**Location:** `filename.ts:42`
**Description:** [what the vulnerability is and how it could be exploited]
**Remediation:** [concrete code change or configuration step]
**Reference:** [OWASP link or CVE ID where applicable]

---

### Axes With No Issues
[list axes that were checked and found clean]
```

---

## Behaviour Constraints

- Do not invent vulnerabilities. If uncertain, state it as a concern to verify.
- Do not rewrite entire files. Provide targeted, minimal remediation snippets.
- Never output secrets found in source code — note their location and instruct rotation.
- If codebase is large, ask user to narrow scope to highest-risk files.

---

## Extended Resources

- `docs/SECURITY_CHECKLIST.md` — printable pre-release checklist for engineers
