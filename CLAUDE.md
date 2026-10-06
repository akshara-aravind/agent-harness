# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **general-purpose agent harness**, not an application. It holds reusable agent definitions, report templates, and per-project context and outputs. It is meant to serve many projects with different needs (migrating code or whole repos, building features, fixing bugs, and work types not yet known). It is in the foundation stage, so keep additions minimal and generic. Don't bake one project's details into shared files.

There is no build, lint, or test tooling: `package.json` is bare `pnpm init` output. `HARNESS_OVERVIEW.md` is the fuller reference.

Agents here investigate and document. The actual work happens in the project's own repositories, never in this repo.

## Layout: separate concerns

- `.claude/agents/<agent>.md`: reusable, project-agnostic subagent definitions (auto-discovered by Claude Code) (frontmatter `name` / `description` / `tools`, then *Configuration*, *Inputs*, *Process*, *Output*, *Hard rules*). Currently only `explorer`.
- `templates/<agent>/`: output structures. `templates/exploration/general.md` (9 sections) is the single report template for all work types, including migrations. Work-type differences go in playbooks, not templates.
- `playbooks/<agent>/<work-type>.md`: extra instructions an agent follows for one work type (for example `playbooks/explorer/migration.md`). The agent's base prompt stays generic. Playbooks live outside `.claude/agents/` so Claude Code doesn't treat them as agents. Add one when a work type needs specific steps, checks or report guidance.
- `projects/<project>/`: everything specific to one project. `project.yaml` lists its `repos` (each with a `role`: source, target, reference or working, and a `link`), `work_types` and `constraints`. `CONTEXT.md` holds domain background. `reports/` holds generated reports as `<task-name>.md` in kebab-case. Start a new project by copying `projects/_TEMPLATE/`.

Current project: `projects/resale-to-seller-migration/`. Its repo `link` values are placeholders (`/path/to/...`) and must be set before an agent can run.

## Adding an agent

Create `.claude/agents/<name>.md`. No registration step is needed. Keep it project-agnostic: project details belong in `projects/`, work-type details in `playbooks/`.

## Cross-file invariants

- `explorer` copies `templates/exploration/general.md`, and its prompt and the playbooks refer to the template path and to section numbers. If you rename or renumber template sections, update `.claude/agents/explorer.md` and the playbooks.
- Every report statement is tagged **[Fact]** (with a citation), **[Assumption]** (with how to verify) or **[Recommendation]** (with reasoning). Missing inputs become blockers in the missing-information section instead of guesses.
- `explorer` may write only `projects/<project>/reports/<task>.md`. Its read-only Bash and no-production-data rules exist only in the prompt. Enforcing them would need `.claude/settings.json` permissions or hooks, which don't exist yet.
- New `project.yaml` keys must be described in `.claude/agents/explorer.md` (*Configuration*) and in `projects/_TEMPLATE/project.yaml`.
