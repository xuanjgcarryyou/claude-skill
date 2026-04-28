# migrating-schema

DB Migration Safety Gate — mandatory approval before any migration file is generated.

## When to Use

Say any of the following to activate:
- "migration"
- "alter table"
- "schema change"
- "add column" / "drop column"
- "schema.prisma" (when mentioned in context of changes)

## What It Does

Produces seven mandatory sections in order, then stops and waits for approval:

1. Impact Analysis — affected tables, code, estimated downtime
2. Risk Explanation — plain-English description of what could go wrong
3. Data Migration Strategy — required for HIGH RISK operations
4. Forward SQL — up migration with transaction wrapper
5. Rollback SQL — down migration (always present)
6. Production Recovery Steps — numbered checklist
7. Approval Gate — outputs and stops

## Risk Classification

| Operation | Risk Level |
|-----------|-----------|
| ADD COLUMN (nullable) | LOW |
| ADD COLUMN (NOT NULL, no default) | **HIGH** |
| ALTER COLUMN (narrowing type) | **HIGH** |
| DROP COLUMN / DROP TABLE | **HIGH** |
| CREATE TABLE | LOW |

## Approval Protocol

- LOW RISK: type `APPROVE` to generate migration files
- HIGH RISK: type `APPROVE HIGH RISK` (not "yes" or "ok") to confirm

## Hard Rules

- No migration file generated before approval
- No up migration without a down migration
- No section can be skipped
- HIGH RISK without explicit data migration strategy → outputs warning and waits
