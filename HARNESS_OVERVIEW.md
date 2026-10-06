# Agent Harness Overview

A general-purpose harness for running agents against many projects. It holds agent definitions, report templates, and per-project context and outputs. It holds no application code. The harness is at the foundation stage: one agent (`explorer`), two templates, and one project.

The design assumes project requirements differ and are not known in advance: migrating code or a whole repo, building features, fixing bugs, and more. So the three things that vary are kept apart:

| Concern | Lives in | Varies by |
|---|---|---|
| **Who does the work** | `.claude/agents/<agent>.md` | Role (explore, plan, implement, review) |
| **Shape of the output** | `templates/<agent>/` | Agent |
| **How a work type changes the job** | `playbooks/<agent>/<work-type>.md` | Work type (migration, feature, fix, ...) |
| **What the work is about** | `projects/<project>/` | Project |

An agent never contains project details. It reads them from `projects/<project>/` at run time.

## Layout

```
agent-harness/
├── CLAUDE.md                    # Guidance for Claude Code sessions
├── HARNESS_OVERVIEW.md          # This document
├── package.json                 # pnpm manifest (no scripts or dependencies yet)
├── .claude/agents/
│   └── explorer.md              # Read-only exploration agent (auto-discovered)
├── playbooks/
│   └── explorer/
│       └── migration.md         # Extra explorer instructions for migration work
├── templates/
│   └── exploration/
│       └── general.md           # 9-section report for all work types
└── projects/
    ├── _TEMPLATE/               # Copy this to start a project
    │   ├── project.yaml
    │   └── CONTEXT.md
    └── resale-to-seller-migration/
        ├── project.yaml         # Repos (source/target), work types, constraints
        ├── CONTEXT.md           # Domain background
        └── reports/             # Generated reports
```

## Projects

`projects/<project>/project.yaml`:

```yaml
name: <project name>
work_types: [migration]          # migration | feature | fix | refactor | investigation | other
summary: <why this project exists>
repos:
  - name: <system>
    role: source                 # source | target | reference | working
    link: <repo path or URL>
constraints: []                  # project-specific rules agents must follow
```

`CONTEXT.md` carries what the code cannot say: goals, domain terms, who can answer questions, decisions already made. Agents read both files first and never edit them. If a location is a placeholder or inaccessible, the agent records a blocker rather than guessing.

To start a project: copy `projects/_TEMPLATE/` to `projects/<name>/`, fill in `project.yaml` and `CONTEXT.md`.

## The explorer agent

Read-only. It investigates a task and writes a report; it never does the work.

1. Reads `project.yaml` and `CONTEXT.md` for the named project.
2. Determines the work type, reads `playbooks/explorer/<work-type>.md` if it exists, and copies `templates/exploration/general.md`, which serves all work types including migrations.
3. Inspects the repos read-only and fills every section, tagging each statement **[Fact]** (cited), **[Assumption]** (with how to verify) or **[Recommendation]** (with reasoning).
4. Writes `projects/<project>/reports/<task-name>.md` and replies with the path, top 3 risks and blocking questions.
5. A person reviews it (Status: Draft, In review, Approved). The approved report is the spec for implementation, which happens elsewhere.

Example request: *"Use the explorer agent on project resale-to-seller-migration: explore migrating seller payout settings."*

Hard rules (prompt-level only): write only the report; no state-changing commands (`prisma migrate`, installs, git commits or pushes, SQL writes); read-only Bash; no production data or secrets.

## Extending

- **New project:** copy `projects/_TEMPLATE/`.
- **New work type:** add `playbooks/explorer/<work-type>.md` (steps, checks, where things go in the report). The agent picks it up from the project's `work_types`; no edit to the agent is needed.
- **New agent** (for example planner, implementer, reviewer): create `.claude/agents/<name>.md` with a specific `description`, a minimal `tools` list and the sections *Configuration*, *Inputs*, *Process*, *Output*, *Hard rules*. Add `templates/<name>/` if it produces structured output. An agent that changes code should work in the project's own repo, with its own tighter rules, not write here.
- **Enforce guardrails:** add `.claude/settings.json` permission denies and a `PreToolUse` hook that limits writes to `projects/*/reports/`.
- **Tooling:** `package.json` can later hold checks, such as that every `project.yaml` has the required keys or every report has all its template sections.

## Design principles

- Separate exploration from execution.
- Evidence over assertion: tag claims.
- Fail visibly: missing inputs become blockers, not guesses.
- Keep agents project-agnostic; put variation in templates and projects.
- Least privilege: give each agent only the tools it needs.
- Stay minimal until a real requirement appears.
