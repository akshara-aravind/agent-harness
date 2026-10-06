# Exploration: <Task Name>

> Copy this file to `projects/<project>/reports/<task-name>.md` (kebab-case) and fill in every section.
> This is an exploration report only. Do not implement or modify any code, schema, configuration, or data.

| Field     | Value                                                  |
| --------- | ------------------------------------------------------ |
| Task      | <short description>                                    |
| Work type | Migration / Feature / Fix / Refactor / Investigation / Other |
| Project   | <project name>                                         |
| Explored by | <name or agent>                                      |
| Date      | <YYYY-MM-DD>                                           |
| Location  | <repo path, branch/commit, environment>                |
| Status    | Draft / In review / Approved                           |

**Labelling convention.** Tag every statement with one of these:

- **[Fact]**: verified directly from code, data, docs or a stakeholder. Cite the source (`path/to/file.ts:42`, query, ticket, person).
- **[Assumption]**: believed true but not verified. Say what would confirm or disprove it.
- **[Recommendation]**: a proposed decision or action, with the reasoning behind it.

---

## 1. Objective
What is wanted and why. In scope, out of scope, success criteria.

## 2. Current State
How the relevant area works today: code, data, APIs, jobs, config, users. Known issues.

## 3. Desired State
What should exist afterwards, and the gap from the current state.

## 4. Affected Areas and Dependencies
| Kind | Name | Location | Notes |
| ---- | ---- | -------- | ----- |
|      |      |          |       |

Upstream dependencies, downstream dependents.

## 5. Reusable vs New
What can be reused or adapted, and what must be built (with size S/M/L).

## 6. Risks and Edge Cases
| #  | Risk / edge case | Likelihood (L/M/H) | Impact (L/M/H) | Mitigation |
| -- | ---------------- | ------------------ | -------------- | ---------- |
| R1 |                  |                    |                |            |

## 7. Missing Information / Questions
| #  | Question | Why it matters | Owner | Status |
| -- | -------- | -------------- | ----- | ------ |
| Q1 |          |                |       | Open   |

## 8. Testing and Validation
How to confirm the work is correct and nothing else broke.

## 9. Recommended Approach
Ordered steps, where the change should live, rollout and rollback, prerequisites (link to section 7), estimated effort.
