# Schema Audit Checklist (Laravel)

Apply each check to every table. Report findings as a table with columns: Table, Issue, Severity (Critical/Warning/Info), Recommendation.

---

## 1. ID Exposure — Sequential IDs on API-Exposed Resources

**Check:** Is this table's primary key exposed in API URLs (`routes/api.php`, controllers) or API response bodies (Resource classes, controller returns)?

**Problem:** Sequential bigint IDs enable enumeration attacks. A client who sees `/tasks/42` can try `/tasks/43` to probe for other users' data. Even with authorization checks, it leaks information (total record count, creation order, activity volume).

**Rule:**
- ID appears in URLs or response bodies to external API consumers → **UUID (v7)**
- ID only used internally (Sanctum tokens, pivot tables, internal lookups behind auth with user-scoped queries) → bigint is fine

**Where to look (Laravel):**
- `routes/api.php` — route parameters like `{task}`, `{template}`
- `app/Http/Controllers/` — IDs returned in responses, used in `find()` or `findOrFail()`
- `app/Http/Resources/` — `$this->id` in `toArray()`

**Fix:** `HasUuids` trait on the model + `$table->uuid('id')->primary()` in migration.

---

## 2. Cascade Delete Safety — Destroying Audit/Billing Data

**Check:** Does any `cascadeOnDelete()` FK destroy records that contain billing, audit, usage, or history data?

**Problem:** Cascade delete is convenient but destructive. If deleting a parent row wipes child rows that contain cost records, API logs, or usage history, that's unrecoverable data loss.

**Rule:**
- Child contains billing/cost/audit data → **nullable FK + nullOnDelete** (orphan the child, preserve data)
- Child is meaningless without parent (e.g., images without a task) → cascade is fine
- Parent is a user-managed resource → consider **soft delete** on the parent instead

**Where to look (Laravel):**
- Migration files — search for `cascadeOnDelete()`
- Check what the child table stores — does it have `cost`, `price`, `amount`, `credits`, `usage`, `log`, `response` columns?

**Fix:** Change `->cascadeOnDelete()` to `->nullable()->nullOnDelete()` on the FK. Or add `SoftDeletes` to the parent model.

---

## 3. Soft Delete Candidates — User-Managed Resources

**Check:** Is this a resource that users create/delete AND that other tables reference via FK?

**Problem:** Hard deleting a user-managed resource (API key, template, project) can either cascade-destroy related data or leave null FKs with no way to recover. Soft delete preserves the record for audit, recovery, and referential context.

**Rule:** Apply `SoftDeletes` when ALL of:
- Users can delete the resource through the API
- Other tables reference it via FK
- Those referencing tables contain data worth preserving

**Don't soft delete:**
- Internal/system tables (providers, ai_models)
- Ephemeral data (cache, jobs, sessions)
- Tables with no inbound FKs

**Where to look (Laravel):**
- Models — does the model have `HasMany` or is it referenced by `BelongsTo` from another model?
- Controllers — is there a `destroy()` method exposed to users?
- Migration FKs — search for `->constrained('this_table')`

**Fix:** Add `SoftDeletes` trait to model + `$table->softDeletes()` in migration.

---

## 4. Index Strategy — Missing Indexes on Query Patterns

**Check:** What are the most frequent query patterns for this table? Are they indexed?

**Problem:** Missing indexes on frequently queried columns cause full table scans. Over-indexing wastes disk and slows writes.

**Rule:**
- Columns in `WHERE` clauses of frequent queries → index
- Columns used in `ORDER BY` on large tables → index
- FK columns → usually auto-indexed by Laravel's `constrained()`, but verify
- Columns only used in rare admin queries → skip

**Where to look (Laravel):**
- Controllers/Services — what `where()`, `orderBy()`, `scope*()` calls exist?
- Model scopes — `scopeActive()`, `scopePending()`, etc.
- Common patterns: status polling, user-scoped listings, date-range queries

**Fix:** `$table->index('column_name')` with a comment explaining the query pattern.

---

## 5. FK Constraint Strategy — On-Delete Behavior

**Check:** For each FK, what should happen when the parent is deleted?

**Problem:** Default FK behavior (`RESTRICT`) blocks parent deletion entirely, which may not be the right UX. Each FK needs an intentional on-delete strategy.

**Decision matrix:**

| Parent deleted | Child data value | Strategy |
|---|---|---|
| Child is meaningless without parent | Low | `cascadeOnDelete()` |
| Child is meaningless without parent | High (audit/billing) | `nullOnDelete()` + nullable FK |
| Child has independent value | Any | `nullOnDelete()` + nullable FK |
| Parent should never be deleted | N/A | `restrictOnDelete()` (default) |

**Where to look (Laravel):**
- Every `foreignId()` / `foreignUuid()` in migrations
- Check if there's an explicit on-delete, if not it defaults to RESTRICT

---

## 6. Unique Constraints — Business Rules the DB Should Enforce

**Check:** Are there uniqueness rules that only application code enforces?

**Problem:** Application-level uniqueness checks are racy under concurrent requests. If two requests check "does this exist?" simultaneously, both may pass and create duplicates.

**Rule:**
- One-per-user resources (one API key name per user, one BYOK key per provider) → `unique(['user_id', 'name'])`
- Natural keys (email, slug, external ID) → `unique('column')`

**Where to look (Laravel):**
- Validation rules with `unique:` — these should have a matching DB constraint
- Business rules like "one X per Y" — need composite unique index
