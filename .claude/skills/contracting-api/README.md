# contracting-api

API Contract Architect — locks interface definitions before parallel frontend/backend development begins.

## When to Use

Say any of the following to activate:
- "define API"
- "lock the interface"
- "API contract"
- "before we implement"
- "frontend-backend boundary"
- "what does the endpoint look like"
- "agree on the shape"

## What It Does

1. Collects: HTTP method, path, consumer, auth mechanism, pagination requirements
2. Outputs a **LOCKED** OpenAPI-compatible YAML contract at v1.0.0
3. Runs a post-contract checklist

## Output Format

```yaml
contract:
  version: v1.0.0
  status: LOCKED
  endpoint:
    method: POST
    path: /api/resource
    auth: Bearer JWT
  request_body:
    required: [...]
    optional: [...]
  responses:
    "201": { body: [...] }
  errors:
    "400": { code: VALIDATION_ERROR }
  changelog:
    - version: v1.0.0
      note: "Initial contract lock"
```

## Version Bump Rules

| Change | Bump |
|--------|------|
| Add optional field | Minor (v1.1.0) |
| Add required field | Major (v2.0.0) |
| Remove any field | Major |
| Change field type | Major |
| Clarify description only | Patch (v1.0.1) |

## Incomplete Input

If details are missing, outputs a `status: DRAFT` contract with `<REQUIRED: description>` placeholders instead of locking.

## Hard Rules

- Never implements endpoints — contract only
- Never outputs without `LOCKED` or `DRAFT` status
- One contract block per endpoint
