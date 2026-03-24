---
name: schema-audit
description: Audit Laravel database schemas for common design issues — UUID for API-exposed IDs, soft deletes for audit/billing data, cascade delete safety, index strategy, FK on-delete behavior, missing unique constraints. Use when designing new tables, reviewing migrations, auditing existing schemas, or when user says "audit schema", "review database", "check migrations", or "schema review". Also use proactively after creating new migration files.
---

# Schema Audit

Audit a Laravel database schema by cross-referencing migrations, models, routes, and controllers against a checklist of common design issues.

## Workflow

### 1. Gather context

Read in parallel:
- `database/migrations/` — all migration files
- `app/Models/` — all model files
- `routes/api.php` — which models are exposed in URLs
- `app/Http/Controllers/` — which IDs appear in responses
- `app/Http/Resources/` — which IDs are serialized to API consumers

### 2. Apply checklist

Load [references/checklist.md](references/checklist.md) and evaluate each table against all 6 checks.

### 3. Report findings

Output a findings table:

| # | Table | Check | Severity | Finding | Recommendation |
|---|-------|-------|----------|---------|----------------|

Severity levels:
- **Critical** — security risk or data loss risk (exposed sequential IDs, cascade deleting billing data)
- **Warning** — design smell that will cause pain later (missing soft delete on referenced resource, missing index on polled column)
- **Info** — minor improvement (missing unique constraint enforceable at DB level)

### 4. Propose changes

For each Critical/Warning finding, show the specific code change needed (migration line + model trait/relationship).

Group by file so changes are easy to apply.

## When everything is clean

If no issues found, confirm with: "Schema audit complete — no issues found across N tables."
