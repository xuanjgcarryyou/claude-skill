# auditing-dependencies

Dependency Health Auditor — seven-point check before any package enters the codebase.

## When to Use

Say any of the following to activate:
- "install" / "npm install" / "yarn add" / "pnpm add"
- "pip install" / "poetry add"
- "add dependency" / "new package"
- "import \<package\>" (when the package is external)
- "should I use \<package name\>"

## What It Does

Runs seven checks and delivers a PASS / WARN / BLOCK verdict:

| Check | BLOCK Condition |
|-------|----------------|
| 1 — Existence | Package not found on registry |
| 2 — Redundancy | Project already has equivalent package |
| 3 — Activity | Repository is archived |
| 4 — Popularity | < 10k weekly downloads (npm) |
| 5 — License | No license, unknown license, or SSPL |
| 6 — Bundle Size | > 500 KB gzipped (frontend packages) |
| 7 — Security | High or Critical CVE |

## Verdict Format

```
DEPENDENCY AUDIT REPORT
────────────────────────
Package:   <name>@<version>

  Check 1 — Existence:    PASS
  Check 2 — Redundancy:   PASS
  ...

FINAL VERDICT: PASS
────────────────────────
Safe to install.
```

Any BLOCK → Final = BLOCK (with alternative recommendation)
Any WARN, no BLOCK → Final = WARN (type PROCEED to override)

## Hallucination Detection

If a package has zero registry results, outputs a `HALLUCINATION RISK` warning with the canonical alternative.

## Missing Manifest

If `package.json` / `requirements.txt` is not provided, Check 2 is marked SKIPPED and the remaining six checks proceed normally.
