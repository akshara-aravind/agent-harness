# Agent Harness Overview

This repository is an **agent harness**. It holds no application code. It holds agent definitions, report templates, and the reports those agents produce.

Right now the harness does one job: **exploring migrations from the Resale Portal (source) to the Seller Portal (target)**. An exploration agent investigates a migration task and writes a structured, read-only report. People then use that report to plan and carry out the migration somewhere else.

---

## 1. Repository Contents

```
agent-harness/
├── HARNESS_OVERVIEW.md          # This document
├── package.json                 # pnpm package manifest (no scripts or dependencies yet)
└── agents/
    ├── exploration.md           # The Migration Exploration agent definition
    └── exploration/
        └── _TEMPLATE.md         # Report template that every exploration report copies
```

| Path | Purpose | Edit by hand? |
|---|---|---|
| `agents/exploration.md` | Defines the `exploration` agent: its name, description, allowed tools and system prompt. | Yes |
| `agents/exploration/` | The exploration agent's working folder. It holds the template and is the intended home for that agent's reports. | Yes |
| `agents/exploration/_TEMPLATE.md` | The canonical 13-section report structure. | Yes, but changes affect every future report (see §6). |
| `agents/exploration/<task-name>.md` | Generated exploration reports, one per migration task. | Yes, for review and refinement after generation |
| `package.json` | pnpm manifest created with `pnpm init` (`pnpm@10.32.1`). It holds only defaults for now. | Yes |

**Convention.** Each agent has a definition file at `agents/<agent-name>.md` and a folder of the same name at `agents/<agent-name>/` for its template and outputs.

**Not present yet.** There is no `CLAUDE.md`, no `.claude/` directory (so no settings, permissions, hooks or skills), no `.mcp.json`, and no completed exploration reports. The directory is also not a git repository. See §7 for how to add these.

> Note: the template file is named `_TEMPLATE.md` (with a leading underscore). The underscore keeps it sorted above the reports and marks it as something other than a report.

---

## 2. How Agents Are Loaded

Agent files use the Claude Code subagent format:

```markdown
---
name: <agent-id>                 # How the agent is referenced
description: <when to use it>    # Claude uses this to decide when to delegate
tools: <comma-separated list>    # Tool allowlist; omit to inherit all tools
---

<system prompt: role, inputs, process, output format, hard rules>
```

> **Important: discovery.** Claude Code only auto-discovers project subagents in `.claude/agents/*.md`. Agents in the top-level `agents/` folder **are not registered automatically**. To use them as Claude Code subagents, choose one of these:
> - Symlink or copy them in, e.g. `mkdir -p .claude && ln -s ../agents .claude/agents`. The `agents/exploration/` subfolder is harmless because only `.md` files define agents. Note that `_TEMPLATE.md` would then also sit inside the agents directory, so check that it isn't picked up as an agent (it has no frontmatter).
> - Or reference the file directly, e.g. *"Follow the instructions in `agents/exploration.md` to explore …"*. This runs the prompt in the main session instead of as a subagent, so the `tools` allowlist is not enforced.

Once registered, the flow is:
1. The `exploration` agent appears in the list of available agent types.
2. The main Claude session delegates to it when a request matches its `description`. You can also ask for it by name.
3. The subagent runs in its own context window, using only the tools in its frontmatter.
4. It writes a report and returns a short summary to the main session.

---

## 3. The Exploration Agent

**File:** `agents/exploration.md`
**Name:** `exploration`
**Tools:** `Read, Grep, Glob, Bash, Write`

### Role
A **read-only Migration Exploration Agent**. It investigates any migration task (data, functionality, DB structures, APIs, configuration, workflows) that moves something from the Resale Portal to the Seller Portal. It **produces a report and never performs the migration.**

### Inputs it expects
- The migration task name and objective.
- The location of the **Resale Portal** (source): repo path, branch, DB or docs.
- The location of the **Seller Portal** (target).

If the source or target is missing or inaccessible, the agent does not guess or substitute unrelated code. It records a blocker in section 11 of the report, fills in what it can, and says clearly what is missing.

### Process (8 steps)
1. Understand what currently exists in the Resale Portal.
2. Identify the source data, code, DB models, APIs, services, jobs, config and workflows involved.
3. Identify the corresponding target structure in the Seller Portal.
4. Compare source and target: schema, types, nullability, enums, relations, behaviour, permissions.
5. Classify each item as reusable, needing transformation or mapping, or needing to be built new.
6. Find dependencies, risks, missing information and edge cases. These include hardcoded IDs, nulls in required columns, orphaned FKs, soft deletes, enum drift, timezones, volume and cut-over.
7. Recommend where and how to implement the migration.
8. Define what to test and validate after migration.

### Output contract
- Follows the template structure exactly, with sections 1–13 under `# Migration Exploration`.
- Writes one report per task, named in **kebab-case**.
- Tags **every statement** as one of:
  - **[Fact]**: verified, with a citation (`file:line`, table or query).
  - **[Assumption]**: unverified, with a note on how to verify it.
  - **[Recommendation]**: a proposed decision, with its reasoning.
- Ends with a short reply giving the **report path**, the **top 3 risks**, and any **blocking open questions**.

> **Path mismatch to fix.** The agent prompt (`agents/exploration.md`) and the template's intro line still use the old paths `exploration/_TEMPLATE.md` and `exploration/<migration-task-name>.md`. Those folders no longer exist at the root. Until the prompt is updated to `agents/exploration/_TEMPLATE.md` and `agents/exploration/<migration-task-name>.md`, the agent will fail to find the template and will create a new top-level `exploration/` folder for its report.

### Safety guardrails (hard rules)
| Rule | Detail |
|---|---|
| Single writable location | The agent may only create or write its report in its report folder. It never edits source code, schemas, migrations, config, `.env` or anything else. |
| No state-changing commands | Banned: `prisma migrate`, `prisma db push`, `prisma db seed`, `npm install`, git commits and pushes, and any `INSERT`/`UPDATE`/`DELETE`/`ALTER`/`DROP`/`TRUNCATE`. |
| Read-only Bash only | Allowed: `ls`, `cat`, `grep`, `find`, `git log`, `git show`, `prisma validate`, and similar. |
| No production data | Never connects to or queries production. Never prints secrets from `.env` files. |

> **Enforcement note.** When the agent runs as a registered subagent, Claude Code enforces the `tools` allowlist: it cannot call `Edit`, `WebFetch` and so on. The "write only to the report folder" and "read-only Bash" rules, however, live in the **prompt**, and `Write` and `Bash` are not path- or command-restricted. For hard guarantees, add permission rules or hooks in `.claude/settings.json` (see §7.4).

---

## 4. The Exploration Report Workflow

```
 ┌──────────────┐    ┌────────────────────┐    ┌───────────────────────┐    ┌──────────────────────┐
 │ User request │ ─▶ │ Main Claude session│ ─▶ │ exploration agent     │ ─▶ │ agents/exploration/  │
 │ task + paths │    │ delegates          │    │ read-only inspection  │    │ <task>.md            │
 └──────────────┘    └────────────────────┘    │ of source & target    │    └──────────┬───────────┘
                                               └───────────────────────┘               │
                                                                                       ▼
                                                      Human review → Status: Draft → In review → Approved
                                                                                       │
                                                                                       ▼
                                                      Implementation happens elsewhere (not in this harness)
```

### Step by step
1. **Request.** Give the task name and objective plus the source and target locations. Example:
   > Use the exploration agent to explore migrating seller payout settings. Resale Portal is at `~/code/resale-portal` (branch `main`); Seller Portal is at `~/code/seller-portal`.
2. **Exploration.** The agent inspects both codebases read-only: models, migrations, services, routes, jobs, config.
3. **Report.** It copies the structure of `agents/exploration/_TEMPLATE.md` into `agents/exploration/seller-payout-settings.md` and fills every section with tagged statements.
4. **Summary.** It returns the path, the top 3 risks and any blocking questions.
5. **Review.** A human checks the report, resolves the open questions in §11, verifies the `[Assumption]` items, and moves the header `Status` field from `Draft` to `In review` to `Approved`.
6. **Hand-off.** The approved report becomes the spec for the actual migration work, which happens in the portal repositories rather than here.

### Report structure (`agents/exploration/_TEMPLATE.md`)

The report opens with a metadata table (Task, Migration type, Explored by, Date, Source location, Target location, Status) and a statement of the labelling convention. Then come the sections:

| # | Section | What it captures |
|---|---|---|
| 1 | Migration Objective | What and why, in and out of scope, success criteria |
| 2 | Source: Resale Portal | Current feature, stack, users, known tech debt |
| 3 | Target: Seller Portal | Corresponding area (or its absence), stack, users, constraints such as multi-tenancy and auth |
| 4 | Source Data / Code / Dependencies | Inventory table of tables, enums, endpoints, services, UI, jobs, integrations, config, storage; upstream and downstream dependencies |
| 5 | Target Structure | Matching inventory with an Exists? (Y/N/Partial) column |
| 6 | Source → Target Mapping | 6.1 entity mapping (cardinality), 6.2 field mapping (types, nullability, Direct/Transform/Default/Derived/Dropped/New), 6.3 API/behaviour gaps |
| 7 | Transformation Requirements | Numbered rules (T1…) with examples and null handling; type conversion, normalisation, ID strategy, FK re-pointing, soft-delete and audit-field semantics |
| 8 | Reusable Components | What can be reused as-is or adapted |
| 9 | New Components Required | What must be built, with size (S/M/L) |
| 10 | Risks and Edge Cases | Risk register (R1…) with likelihood, impact and mitigation, plus a 12-item checklist |
| 11 | Missing Information / Questions | Open questions (Q1…) with owner and status; blockers go here |
| 12 | Testing and Validation | Pre-migration profiling, reconciliation, integrity, functional, permission, performance, idempotency and rollback tests, sign-off |
| 13 | Recommended Migration Approach | Strategy (big-bang, phased, dual-write, backfill), where the code should live, ordered steps, cut-over and rollback, prerequisites, effort |

### Existing reports
There are none yet. `agents/exploration/` holds only `_TEMPLATE.md`.

---

## 5. Design Principles

- **Separate exploration from execution.** Agents here investigate and document. They never change the systems they look at.
- **Evidence over assertion.** The `[Fact]` / `[Assumption]` / `[Recommendation]` tagging makes clear how much each claim can be trusted.
- **Fail visibly, never silently.** Missing inputs become blockers in §11 instead of guesses.
- **One template, consistent output.** Every report has the same 13 sections, so reports are easy to compare and review.
- **Least privilege.** Each agent gets only the tools it needs.
- **One folder per agent.** An agent's definition (`agents/<name>.md`) sits next to its working folder (`agents/<name>/`).

---

## 6. Maintaining the Template

- Change `agents/exploration/_TEMPLATE.md` when **every** future report needs the change. The agent copies it "exactly", so changes apply automatically.
- If you renumber or rename sections, update the references in `agents/exploration.md` too. For example, the agent prompt names "section 11" as the place for blockers and "sections 1 to 13".
- If you move the template or report folder again, update the paths in `agents/exploration.md` and in the template's own intro line.
- Existing reports are not regenerated automatically. Update them by hand or re-run the agent.

---

## 7. Extending the Harness

### 7.1 Add a new agent
1. Create `agents/<agent-name>.md` with frontmatter and a prompt.
2. Create `agents/<agent-name>/` with a `_TEMPLATE.md` for its output.
3. Make sure it is registered with Claude Code (see §2).

```markdown
---
name: implementation-planner
description: Turns an approved exploration report into a step-by-step implementation plan at agents/implementation-planner/<task>.md. Use after a report in agents/exploration/ is marked Approved.
tools: Read, Grep, Glob, Write
---

You are the Implementation Planning Agent. ...
## Inputs
## Process
## Output
## Hard rules
```

Guidelines, following the pattern of the existing agent:
- Write a **specific `description`**, because it decides when Claude delegates to the agent.
- Keep the **tool list minimal**. Leave out `Write` and `Bash` unless the agent truly needs them.
- Use **full paths from the repo root** (`agents/<name>/…`) in the prompt.
- Use the same body sections: *Inputs*, *Process*, *Output*, *Hard rules*.

Ideas that fit the current workflow:
| Agent | Input | Output |
|---|---|---|
| `report-reviewer` | An exploration report | Review comments: untagged statements, missing citations, empty sections |
| `implementation-planner` | An approved report | `agents/implementation-planner/<task>.md` |
| `validation-script-writer` | Report §12 | Read-only reconciliation SQL and checks |
| `migration-index` | All reports | `agents/exploration/INDEX.md` listing each task, its status and top risks |

### 7.2 Add project context (`CLAUDE.md`)
A root `CLAUDE.md` loads into every session. Good contents:
- Default paths to the Resale Portal and Seller Portal repos, so users don't have to repeat them.
- The `agents/<name>.md` + `agents/<name>/` folder convention.
- Glossary of domain terms (seller, resale, listing and so on).
- Naming conventions for reports.

### 7.3 Use `package.json` for tooling
`package.json` currently holds only `pnpm init` defaults. Useful additions:
- `"private": true`, to prevent accidental publishing. Remove the unused `"main": "index.js"`.
- Scripts such as `lint:md` (markdownlint or Prettier over `agents/**/*.md`) or a check that every report has all 13 sections and a filled-in metadata table.

### 7.4 Enforce guardrails with settings and hooks
Create `.claude/settings.json` to turn prompt rules into enforced rules, for example:
- **Permissions:** `deny` rules for `Bash(prisma migrate:*)`, `Bash(npm install:*)`, `Bash(git push:*)` and `Read(**/.env*)`.
- **Hooks:** a `PreToolUse` hook on `Write` that blocks paths outside `agents/*/`, or a hook that flags SQL write keywords in Bash commands.

The built-in `update-config` skill can generate these settings.

### 7.5 Add slash commands or skills
Add skills under `.claude/skills/<name>/SKILL.md` for repeated entry points. For example, `/explore <task-name>` could prompt for source and target paths and then delegate to the `exploration` agent.

### 7.6 Add MCP servers
If agents need read-only database access or ticket lookups, configure them in `.mcp.json` and add the specific MCP tool names to each agent's `tools` list. Point any database access at non-production replicas only, in line with the existing hard rules.

### 7.7 Version control
The directory is not yet a git repository. Running `git init` lets you track changes to agents, templates and reports, and review report revisions. Add `node_modules/` and `.claude/scheduled_tasks.lock` (a Claude Code runtime file) to `.gitignore`.

---

## 8. Quick Reference

| I want to… | Do this |
|---|---|
| Explore a migration | Ask Claude: *"Use the exploration agent to explore `<task>`; source at `<path>`, target at `<path>`."* |
| Find a report | `agents/exploration/<kebab-case-task-name>.md` |
| Change report structure | Edit `agents/exploration/_TEMPLATE.md` and keep the section references in the agent prompt in sync |
| Change agent behaviour | Edit `agents/exploration.md` |
| Add an agent | Create `agents/<name>.md` (frontmatter + prompt) and `agents/<name>/_TEMPLATE.md`, then register it (§2) |
| Enforce safety rules | Add `.claude/settings.json` permissions and hooks |
