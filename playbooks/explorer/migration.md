# Explorer playbook: migration

Extra instructions for the explorer agent when the work type is **migration** (moving data, code, features, schemas, APIs, configuration, workflows or a whole repo from a source system to a target system). These add to the base agent prompt and the report template. They never relax the agent's hard rules.

## Inputs to confirm
- Source and target locations: both must come from `project.yaml` (repos with role `source` and `target`) or the request. If either is missing, a placeholder or inaccessible, record a blocker; do not guess.
- What exactly moves (one feature, a data set, the whole repo) and whether the source stays live during the move.

## What to investigate
1. The source: data models, code, APIs, services, jobs, config, storage and integrations involved, plus what depends on them (reports, other services, webhooks, mobile apps).
2. The matching target structure, and whether it exists, partly exists or must be built.
3. A source-to-target comparison: schema, types, nullability, enums, relations, behaviour, permissions, naming and validation rules, multi-tenancy and auth model.
4. Classify every item: reuse as-is, transform or map, build new, or drop.

## Where to put it in the report
- Sections 2 and 3: source and target, including target constraints.
- Section 4: inventory and dependencies. Put field-level mapping tables here: source field and type, target field and type, mapping kind (direct, transform, default, derived, dropped, new), notes.
- Section 5: reuse versus new.
- Section 6: the risk list below.
- Section 8: the validation list below.
- Section 9: strategy (big-bang, phased, dual-write, backfill), where the code should live (for example a schema migration, a separate idempotent data script, an adapter layer), ordered steps, cut-over and rollback.

## Transformation rules
For each transformation, state the rule, an example (in and out) and how invalid or null values are handled. Cover type conversions, value normalisation (casing, units, currency, formats), ID strategy (keep source IDs, generate new ones or keep a legacy-ID cross-reference), foreign-key re-pointing, soft-delete and archive semantics, and audit fields (preserve or reset).

## Risks to check
- Nulls in source columns that are required in the target
- Duplicates or uniqueness conflicts in the target
- Orphaned references (foreign keys pointing to missing users, sellers or entities)
- Enum or status values present in data but missing from the target enum
- Soft-deleted or trashed records: migrate, skip or archive
- Hardcoded IDs or environment-specific data in existing migrations or seeds
- Data volume, batch size, locking and downtime
- Timezone and precision loss
- Ownership and permission differences between the systems
- Records changed in the source during migration (cut-over and delta sync)
- PII and compliance handling
- Rollback feasibility

## Validation to define
- Pre-migration: data profiling (row counts, null rates, distinct enum values, orphan checks).
- Reconciliation: source versus target row counts per entity, checksums or aggregates on key numeric fields, field-by-field spot checks of sampled records.
- Integrity: foreign keys, uniqueness, required fields, enum validity.
- Functional and permissions: the target behaves like the source for key user journeys, and the right users see the right records.
- Performance: runtime on production-sized data.
- Idempotency and rollback: re-running is safe, rollback has been rehearsed.
