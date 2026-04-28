# Pre-Release Security Checklist

Printable checklist for engineers before any high-risk feature ships to production.
Run the `reviewing-security` skill first, then use this as a final gate.

---

## How to Use

1. Run `/reviewing-security` on the relevant code changes.
2. Resolve all CRITICAL findings before proceeding.
3. Work through this checklist top to bottom.
4. All items must be checked before the PR can be merged to production.

---

## Axis 1 — Injection

- [ ] All database queries use parameterized statements (no string concatenation)
- [ ] No user input passed to `exec()`, `spawn()`, `system()`, or shell commands
- [ ] All user-controlled values rendered in HTML are escaped (no `dangerouslySetInnerHTML` without sanitization)
- [ ] LDAP filter strings do not contain unescaped user input

## Axis 2 — Authentication

- [ ] Passwords are hashed with bcrypt (cost ≥ 12), argon2id, or scrypt
- [ ] MD5 / SHA-1 / SHA-256 are NOT used for password storage
- [ ] Session tokens have ≥ 128 bits of entropy
- [ ] Session cookies have `HttpOnly` and `Secure` flags set
- [ ] Sessions are invalidated on logout
- [ ] JWT algorithm is pinned server-side (no `alg: none` accepted)
- [ ] JWT expiry is enforced

## Axis 3 — Authorization

- [ ] Resource IDs from user input are validated against ownership before serving
- [ ] Admin-only endpoints have role checks at the middleware layer
- [ ] Authorization middleware is applied to every route that reads or mutates sensitive data

## Axis 4 — Sensitive Data Exposure

- [ ] PII (email, phone, SSN, IP) is not written to logs
- [ ] API responses do not return fields the caller does not need
- [ ] All endpoints serving sensitive data require HTTPS
- [ ] No password or secret is echoed in any response body

## Axis 5 — Misconfiguration

- [ ] `Access-Control-Allow-Origin: *` is NOT combined with `credentials: true`
- [ ] `Content-Security-Policy` header is present
- [ ] `Strict-Transport-Security` header is present
- [ ] `X-Frame-Options` header is present
- [ ] `X-Content-Type-Options: nosniff` header is present
- [ ] Error handlers do NOT return stack traces or database schema details to clients
- [ ] Debug mode is disabled in production configuration

## Axis 6 — CSRF / SSRF

- [ ] All state-mutating endpoints validate a CSRF token
- [ ] Server-side HTTP requests built from user-supplied URLs use an allowlist
- [ ] No open redirect — redirected URLs are validated before use

## Axis 7 — Secrets

- [ ] No hardcoded API keys, tokens, or passwords in source files
- [ ] `.env` files are listed in `.gitignore`
- [ ] Secrets are not visible in Docker build arguments
- [ ] CI logs do not print environment variable values

## Axis 8 — Input Validation

- [ ] All user input is validated at the system boundary (controller/handler layer)
- [ ] String fields have maximum length constraints
- [ ] File uploads validate MIME type and file size
- [ ] Numeric inputs have range constraints

## Axis 9 — Dependencies

- [ ] No dependency in the current version range has a known High or Critical CVE
- [ ] Security-sensitive packages (auth, crypto, parsing) are at their latest stable release
- [ ] No abandoned packages are used in security-critical paths

---

## Ship Gate Decision

| Findings | Decision |
|----------|----------|
| Any CRITICAL | BLOCKED — must fix before merge |
| Any HIGH | Fix in current sprint before production deploy |
| MEDIUM only | Merge permitted — schedule fix in next version |
| LOW only | Merge permitted — track as tech debt |
| None | Clear to ship |

---

## Reference

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
