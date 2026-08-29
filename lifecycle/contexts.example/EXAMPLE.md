# Lifecycle adapter: example-org/example-api

Copy this file to `../contexts/<repo>.md`, where `<repo>` is the basename of
`git remote get-url origin`, and replace every value. The engine skills carry no
repository facts at all — everything they need to touch your codebase resolves
through this table, so a wrong value here is the only way they can misfire.

Resolution order (LIFECYCLE.md § The adapter):
`.claude/lifecycle-context.md` in the repo → `~/.claude/skills/lifecycle/contexts/<repo>.md` → bootstrap.

| Key | Value |
|---|---|
| `{plan_store}` | `~/.claude/lifecycle-plans/` — a git repo of its own, outside every product repo. Plans are process, not deliverable; they reach reviewers through the PR description |
| `{plan_naming}` | `YYYYMMDD-HHMMSS-example-api-<slug>-plan.md` (repo in the filename, since the store spans repos) |
| `{conventions_files}` | `AGENTS.md`, `docs/conventions.md` — read before anything else; AGENTS.md overrides |
| `{test_command}` | `make test` |
| `{test_focus_command}` | `make test FILE=path/to/file` or `FILE:LINE` |
| `{lint_command}` | `make lint` over changed files |
| `{ci_command}` | `make ci` — the local everything-gate. Omit the key if the repo has none |
| `{run_server}` | `make dev` (port 8080) |
| `{workspace_bootstrap}` | `make setup && make db-prepare` — once per fresh clone or worktree |
| `{mint_test_state}` | `bin/seed-user <traits...>` — puts the system into a named state so `[manual]` criteria are checkable. No such tool → say so; verification degrades to a cold start |
| `{manual_probe}` | `http://localhost:8080/api` with a bearer token the seeder prints; admin UI at `/admin` |
| `{regen_artifacts}` | On behavior change: `make openapi` (never hand-edit `openapi/**`); `make migrate` re-annotates models |
| `{followup_routing}` | `TODO.md`, or the team's issue tracker — name the destination and the entry format |
| `{diff_audit_skill}` | The repo's pre-PR self-audit skill, if it has one. Absent → build performs the generic cull inline |
| `{review_skill}` | The repo's PR-comment adjudication skill, if it has one. Absent → the generic description applies |
| `{tracker_location}` | Where lesson rows live — a database, a checked-in CSV, anything queryable. Optional, never a dependency |
| `{invariants}` | Facts no agent may re-derive wrong: money as integer cents, never log raw PII, every error response uses one envelope, seeds are idempotent |
| `{companion_repos}` | See below. Omit the section when the repo's behavior spans nothing |

## Companion repositories

Only repos whose behavior this one's contract genuinely spans. Writing to one
requires it to be named in a plan's `## Companion repos` and approved at gate 2;
reading any repo is always free.

| Repo | Relationship | Deploy order | Notes |
|---|---|---|---|
| `example-org/example-client` | **Consumer** of this API's contract | API first (additive), client after | Its client code is generated from this repo's OpenAPI, so a contract change means regenerating there |
| `example-org/example-upstream` | **Producer** this service consumes via a published package | upstream → package release → version bump here | "Merged upstream" is not "available here" — confirm the release, not the merge |

Local clones live at `~/src/<repo>`. Never write into another session's worktree.

## Repo texture worth knowing at plan time

Short, high-leverage facts a planner would otherwise learn the expensive way:
the architectural pattern new work should follow, the generated files that must
never be hand-edited, the flag system, whether the product has live users, and
one or two exemplar files worth imitating. This section is where repo knowledge
accumulates — the engine stays generic precisely because this file gets specific.
