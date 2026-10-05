# Migration Exploration: <Migration Task Name>

> Copy this file to `exploration/<migration-task-name>.md` (kebab-case, e.g. `Seller-listings-data.md`) and fill in every section.
> This is an exploration report only. Do not implement, migrate, or modify any code, schema, configuration, or data.

| Field           | Value                                                                  |
| --------------- | ---------------------------------------------------------------------- |
| Task            | <short description of what is being migrated>                          |
| Migration type  | Data / Schema / API / Functionality / Configuration / Workflow / Mixed |
| Explored by     | <name or agent>                                                        |
| Date            | <YYYY-MM-DD>                                                           |
| Source location | <repo path, branch/commit, DB, environment>                            |
| Target location | <repo path, branch/commit, DB, environment>                            |
| Status          | Draft / In review / Approved                                           |

**Labelling convention.** Tag every statement with one of these:

- **[Fact]**: verified directly from code, schema, data, docs or a stakeholder. Cite the source (`path/to/file.ts:42`, table name, query, ticket, person).
- **[Assumption]**: believed true but not verified. Say what would confirm or disprove it.
- **[Recommendation]**: a proposed decision or action, with the reasoning behind it.

---

# Migration Exploration

## 1. Migration Objective

- What is being migrated, and why. **[Fact]/[Assumption]**
- In scope:
- Out of scope:
- Success criteria (what "done and correct" means):

## 2. Source: Resale Portal

- Overview of the feature, data, or workflow as it exists today.
- Tech stack involved (framework, ORM, database, external services).
- Who uses it (roles and user journeys) and how often.
- Current known issues or technical debt.

## 3. Target: Seller Portal

- Overview of the corresponding area in the Seller Portal, or a note that it does not exist yet.
- Tech stack involved.
- Who will use it (roles and user journeys).
- Constraints the target imposes (conventions, multi-tenancy, auth model, naming, validation).

## 4. Source Data / Code / Dependencies

| Kind                            | Name | Location | Notes                   |
| ------------------------------- | ---- | -------- | ----------------------- |
| DB table / model                |      |          | row count, owner        |
| Enum / lookup                   |      |          |                         |
| API endpoint                    |      |          | method, auth, consumers |
| Service / module                |      |          |                         |
| UI page / component             |      |          |                         |
| Background job / cron / queue   |      |          |                         |
| External integration            |      |          |                         |
| Config / env var / feature flag |      |          |                         |
| File / blob storage             |      |          |                         |

Upstream dependencies (what the source relies on):

Downstream dependents (what relies on the source: reports, other services, webhooks, mobile apps):

## 5. Target Structure

| Kind                | Name | Location | Exists? (Y/N/Partial) | Notes |
| ------------------- | ---- | -------- | --------------------- | ----- |
| DB table / model    |      |          |                       |       |
| API endpoint        |      |          |                       |       |
| Service / module    |      |          |                       |       |
| UI page / component |      |          |                       |       |
| Job / integration   |      |          |                       |       |

## 6. Source → Target Mapping

### 6.1 Entity / table mapping

| Source entity | Target entity | Cardinality (1:1, 1:N, N:1, split, merge) | Notes |
| ------------- | ------------- | ----------------------------------------- | ----- |
|               |               |                                           |       |

### 6.2 Field mapping

| Source field | Source type / nullability | Target field | Target type / nullability | Mapping (Direct / Transform / Default / Derived / Dropped / New) | Notes |
| ------------ | ------------------------- | ------------ | ------------------------- | ---------------------------------------------------------------- | ----- |
|              |                           |              |                           |                                                                  |       |

### 6.3 API / functionality mapping

| Source endpoint or behaviour | Target endpoint or behaviour | Gap / difference |
| ---------------------------- | ---------------------------- | ---------------- |
|                              |                              |                  |

## 7. Transformation Requirements

For each transformation, give the rule, an example input and output, and how invalid values are handled.

| #   | Field(s) | Rule | Example (in → out) | Invalid / null handling |
| --- | -------- | ---- | ------------------ | ----------------------- |
| T1  |          |      |                    |                         |

Also cover:

- Type conversions (string to enum, float to decimal, timestamp to timezone-aware).
- Value normalisation (casing, trimming, units, currency, phone and plate formats).
- ID strategy: keep source IDs, generate new ones, or keep a legacy-ID cross-reference.
- Relationship and foreign-key re-pointing (users, Sellers, tenants).
- Soft-delete, trash, and archive semantics.
- Audit fields (created/updated/deleted by and at): preserve or reset?

## 8. Reusable Components

| Component | Source location | Reuse as-is / Adapt | Notes |
| --------- | --------------- | ------------------- | ----- |
|           |                 |                     |       |

## 9. New Components Required

| Component | Type (schema, migration script, API, UI, job, config) | Purpose | Est. size (S/M/L) |
| --------- | ----------------------------------------------------- | ------- | ----------------- |
|           |                                                       |         |                   |

## 10. Risks and Edge Cases

| #   | Risk / edge case | Likelihood (L/M/H) | Impact (L/M/H) | Mitigation |
| --- | ---------------- | ------------------ | -------------- | ---------- |
| R1  |                  |                    |                |            |

Checklist to consider:

- [ ] Nulls in source columns that are required in the target
- [ ] Duplicates or uniqueness conflicts in the target
- [ ] Orphaned references (FKs pointing to missing users, sellers, or entities)
- [ ] Enum or status values present in data but missing from the target enum
- [ ] Soft-deleted or trashed records: migrate, skip, or archive?
- [ ] Hardcoded IDs or environment-specific data in existing migrations or seeds
- [ ] Data volume, batch size, locking, and downtime
- [ ] Timezone and precision loss
- [ ] Ownership and permission differences between the portals
- [ ] Records changed in the source during migration (cut-over and delta sync)
- [ ] PII and compliance handling
- [ ] Rollback feasibility

## 11. Missing Information / Questions

| #   | Question | Why it matters | Owner / who can answer | Status |
| --- | -------- | -------------- | ---------------------- | ------ |
| Q1  |          |                |                        | Open   |

## 12. Testing and Validation Requirements

- **Pre-migration:** data profiling (row counts, null rates, distinct enum values, orphan checks).
- **Reconciliation:** source vs target row counts per entity; checksums or aggregates on key numeric fields; spot-check N sampled records field by field.
- **Integrity:** FK validity, uniqueness, required fields, enum validity.
- **Functional:** the target APIs and UI behave like the source for key user journeys.
- **Permissions:** the right users and sellers see the right records.
- **Performance:** migration runtime on production-sized data; target query performance.
- **Idempotency and rollback:** re-running is safe; rollback has been rehearsed.
- **Sign-off:** who validates, and the acceptance criteria.

| Check | Method / query | Expected result |
| ----- | -------------- | --------------- |
|       |                |                 |

## 13. Recommended Migration Approach

- **[Recommendation]** Strategy: big-bang / phased / dual-write / backfill and cut-over.
- Where it should live (for example a Prisma migration for schema, a separate idempotent script for data, an API adapter layer).
- Ordered steps:
  1.
  2.
  3.
- Cut-over and rollback plan:
- Prerequisites and blockers (link to section 11):
- Estimated effort:
