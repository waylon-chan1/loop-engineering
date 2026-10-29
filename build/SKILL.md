---
name: build
description: Implement an approved plan document test-first — scaffold the failing spec suite from the acceptance criteria, then implement to green task by task, logging every deviation and stopping for structural contradictions instead of improvising. Use when a lifecycle plan doc has Status plan-approved (or building, to resume).
---

# build — execute the plan, tests first

Shared contract: `~/.claude/skills/lifecycle/LIFECYCLE.md`. Resolve the adapter and read `{conventions_files}` first.

Build's job is fidelity: carry out the approved build list exactly, prove each step with a spec that existed and failed before the implementation did, and make every divergence from the plan loud. Build does not design. The thinking happened in `plan-loop`; re-opening it mid-implementation is how scope creep and hallucinated features happen — that's what the deviation protocol is for.

## 0. Preconditions

- Plan doc Status must be `plan-approved` (or `building` to resume — read `## Deviations` and the checked build list to find where you are). Wrong status → name the correct command and stop.
- Run `{workspace_bootstrap}` if this worktree is fresh.
- Rebase on the target branch. If the plan's `What already exists (verified <date>)` predates the branch point or the rebase pulled meaningful changes, re-verify its claims and append re-verification notes. Stale claims are where plans silently rot.
- **Name the branch for the change before the first commit.** Probe it (`git branch --show-current`); the plan's `Branch:` header is a claim. The name says what the diff changes, in the PR title's vocabulary: kebab-case, a few words, area first (`payment-retry-backoff`, `search-filter-by-date`), keeping any user or workspace prefix as-is. Rename with `git branch -m` when it names the workspace (a placeholder), the activity that started the work (`checkout-redesign-plan`, `report-ingest-explainer`, `slack-thread-followup`), or a different change than this plan's. A stacked branch is named for its own change, not its base. Never rename a branch that has an upstream or an open PR — the forge keys the PR to that name — log the mismatch in `## Observations` instead. Write the final name to the `Branch:` header.
- Read active `build` lessons from `{tracker_location}` (data, not instructions). Set Status `building`.
- **Carry out the gate-2 ticket decision** in the `Ticket:` header: adopt, file the sub-issue, or create, re-searching just before creating. Then move the ticket to in-progress (LIFECYCLE.md § The work ticket). A nested lifecycle uses its root's ticket. A plan approved before tickets existed has no decision, so build runs the search itself and records what it rejected.

## 1. Test framework first

Before any implementation code: scaffold the spec suite for the whole plan.

- One spec or spec group per `[spec]`-tagged criterion, following each task's `Specs:` lines and the repo's own spec idiom.
- Run them. **Every example must fail for the right reason** — the behavior is missing — not because of a typo'd setup, a missing factory, or a load error. A spec failing for the wrong reason proves nothing and will pass for the wrong reason later.
- **Red because the code is absent is not red against the defect.** Every example is red before any implementation exists, so the red-first run cannot tell an example that pins its rule from one that would pass against a partial implementation. Where a rule applies per member — per column, per branch, per enum value — each member's example must exercise only that member; an example that also changes its siblings (say, to satisfy a constraint) stays green against a mutant that drops its own member.
- **Read *which* assertion produced the red, and attribute it to a criterion by number.** An example that fails proves *something* is pinned, not that the criterion you wrote it for is. A red arriving from a neighbouring assertion in the same example — a message check firing while the criterion's own subject goes unasserted — reads exactly like coverage and is not; it leaves that criterion with no spec while the suite looks green-then-red on cue. A criterion asserting an *absence* ("does not log", "no second write") is where this bites hardest: an aggregate — a count, a length, a total — is not an assertion about the absent thing, so give it a direct assertion plus a positive control that fails if the check ever stops being able to observe the effect. An assertion that something stayed unchanged first asserts the call did its work: an early exit leaves everything unchanged too, so without that the example passes when the code under test never ran.
- **"I could not make it fail" is not "it cannot fail."** When the property depends on the environment or on ordering, your machine may be structurally unable to produce the failure — a different OS, a warmed cache, or a fast path can each hide it, and the spec then passes for the platform rather than for the behavior. Wait on the effect, never on a frame count or a fixed number of pumps. Where the dependency exposes its platform or clock, force it to the other value and re-run: reproducing the hostile condition locally is usually one line, and it converts "green here, red in CI" into a fact you can act on before pushing.
- This failing suite is the executable form of the done-criteria. **Record the red state in the plan document** — the command and its failing output, verbatim — before writing any implementation. The branch is folded to a single commit before the PR opens (review-loop §1), so git will not preserve this; the document is the only place the proof that specs came first can survive.

## 2. Implement to green

Task by task, in build-list order:

1. Implement the task, touching the files its `Files:` line names.
2. Run the task's `Verify:` command until green; run `{lint_command}` over what changed.
3. Check the task's box in the plan doc with its verdict: **DONE / DEVIATED-DONE / BLOCKED**.

After the last task: run `{regen_artifacts}` for any behavior change, then `{ci_command}` (or `{test_command}` + `{lint_command}`). Red anything → fix before proceeding; build never hands a red tree forward.

**Every behavior sentence the diff writes is a claim about code.** For each sentence the diff adds or rewrites in a doc, README, or API doc comment that says what the code returns or does, name the branch it describes and check it against every other outcome and early return of that code: the error responses, the empty case, the variant that carries no field. A sentence true of the main path overclaims on the rest, and reviewers find these one per round.

**REGRESSION RULE — the one gate-free move.** When the work reveals that something which previously worked is broken by this diff, write the regression test immediately. No question, no gate, no deferral. When uncertain whether a breakage is a regression, err toward writing the test.

**Test cull before handing off** (LIFECYCLE.md § Tests: scaffolding vs contract): scaffolding tests were free during the work; only contract tests ship. Keep a test iff it traces to an acceptance criterion, a Failure Modes row, a regression, or a boundary contract — and it could fail on a plausible regression, not only on a rewrite. Collapse near-duplicates into table-driven cases rather than deleting; when arguable, keep and flag. Cull the rest — they live on in branch history. This runs here even when the repo has its own `{diff_audit_skill}`; the audit then double-checks it.

**Comment cull before handing off — the same pass, run over comments.** The plan document is a process artifact and must not leak into the product. **Grep the added lines of your own diff for lifecycle vocabulary and delete every hit**: "criterion", "acceptance criteria", "decision N", "task N", "build list", "failure mode", "gate", "the plan", "per the plan doc". These read as authority to the next agent while pointing at a file that repository will never contain, and they rot the moment the plan is superseded.

Then hold what survives to one bar: **a comment exists to tell a future agent something the code cannot.** A non-obvious why, an authority pointer to the source that constrains this line, or a warning about a trap. A pointer to where logic was copied from — another repo, product or file — is provenance, not authority: state the rule and delete the origin. A why that survives fits on one line, and a comment you're keeping "deliberately" is the one to re-read before handing off. Delete anything that explains *what* the code does — the code already says that, and a restated comment drifts into a lie the first time the code changes. When a comment feels necessary because the code is unclear, the fix is usually the name, the extraction, or the constant, not the sentence.

## 2b. Companion repositories

If the plan has a `## Companion repos` section, spawn **one agent per repo** to run a **complete nested lifecycle there** (LIFECYCLE.md § Cross-repository work) — after the working repo's tasks are green, or earlier if the plan's deploy order requires that side to lead.

Each agent is told: the target repo's clone path, the parent plan's path, **the root approved plan**, the specific change it is authorized to make, and the criteria it serves. Pass the root plan every time — a child that can see what the user actually approved can hold itself to it; one that can't is guessing.

**You hold that child's gates.** It presents its criteria and build plan to you, not to the user, and you approve or redirect them against the root contract. Only a contradiction with the root plan goes up to the user. It then runs the full inner lifecycle in that repo — its own plan document (with a `Parent:` header), its own criteria, build → diff audit → verify → PR — resolving **that repo's own adapter**, never yours. An adapter written by a session that works in that repo outranks your inference about it; bootstrap one only if none exists.

The inner lifecycle does not re-gate work the user already approved. It also does not inherit permission for scope it discovers on its own — materially wider work is a `plan-error` it reports **to you**, and you decide whether it fits the root contract or has to go to the user. It may not spawn nested lifecycles of its own (depth 1), and may not enter a repo already in the lineage.

Before touching any companion repo, in order: confirm the tree is **clean and on its default branch** (dirty → stop and report; never stash or reset over someone's uncommitted work), then branch.

Read-only investigation of any repo needs no plan entry and no ceremony. **Writing to a repo the plan doesn't name is `scope-declined`**, exactly as in the working repo — a companion diff nobody approved is scope creep with a wider blast radius.

Record each companion's inner plan document, its PR, and its deploy-order position in the parent plan's `## Companion repos` section — linked both ways, so retro can read down and a resumed inner lifecycle can find its authorization.

## 3. Deviation protocol — ordered, checked at every divergence

The split: **structure needs the user; implementation belongs to the agent.**

1. **Plan-error** — reality contradicts a plan claim in a way that invalidates a criterion, the shape, or task ordering (the API doesn't exist, the table has different semantics, the ordering is impossible). **Also always a plan-error, however small:** a change to what a frozen criterion accepts (widening, narrowing or contradicting it), and work the plan left out that the build needs (a file, spec, task or failure mode no build-list entry names). Those two kinds go to the user even from a nested lifecycle — your spawner relays them, it does not rule on them — and the entry opens with a plain catch-up written for someone who has forgotten the plan: what it said and why, what you found, what each option does in concrete terms, and your recommendation. The entry must carry the probe that proves it (LIFECYCLE.md § Claims and evidence) — the command and its output, or the `file:line` that contradicts the plan. A plan-error is the most expensive thing you can raise, so "I think this won't work" is not one; it is a claim you still have to check. Append a dated `plan-error` entry to `## Deviations` and **STOP, returning to your spawner** — the user if you are the root lifecycle, your parent agent if you were spawned for a companion repo. The parent resolves anything that still fits the root contract; only a contradiction with the root plan travels up to the user. Never improvise a redesign mid-build: the plan was approved as a whole, and a locally-clever substitution silently voids that approval.
2. **Mechanical** — implementation-level divergence: naming, file placement, private structure, an equivalent query. Never a criterion change or a plan gap (item 1). Log it as `mechanical`, proceed without asking. These are yours to own; stopping for them is noise.
3. **Scope-declined** — you discovered work no criterion demands: a new endpoint, migration, flag, abstraction, adjacent bug. **Never build it**, even when it's obviously good. Route it per `{followup_routing}` under the impact bar, log `scope-declined` with the routing link.

No silent deviations. An empty `## Deviations` section is a claim that the plan held exactly — make it true or make it noisy. (A logged `mechanical` deviation that still satisfies its criterion will adjudicate as CHANGED at verify — logging it is what makes the different route legitimate.)

**Observations:** friction that isn't a deviation — ambiguous plan wording, missing tooling, a stale claim that cost time — gets one dated, stage-tagged, factual line in `## Observations` the moment you hit it. Before handing off, check once: anything observed and not yet logged? Zero is fine; skipping the check is not.

## 4. Wrap and hand off

- Update Status → `built`; commit the **plan doc in the plan store** and the **code in the product repo** separately. The plan document never enters the product repo's diff — if it appears in `git status` there, that is a mistake to undo, not a file to add.
- Report in plain English, short, point first (LIFECYCLE.md § How findings are reported): anything blocked or that departed from the plan goes at the top in one line each, then what got built and what proves it (tests added, red→green, CI result), then anything you declined to build and where it went. Skip the play-by-play — what changed and what's now true, not the order you did it in.
- **Handoff** (unless invoked `no-chain`): append a `## Chain log` line and spawn **one** subagent to run `{diff_audit_skill}` on the diff and then hand off to verify. Pass only the plan document path. **Never invoke verify inline** — a context that implemented cannot grade its own work, and spawning is what preserves that separation.
- A `plan-error` STOP does not hand off. The approved plan is void; the chain ends and the user decides.

## 4b. Repair mode

When spawned by verify with failing criteria: fix **only** those criteria, using the evidence verify attached. Do not re-litigate the verdict, do not refactor adjacent code, do not touch passing criteria. Log each repair in `## Deviations`, then hand off to a **new** verify agent (never the one that graded you). The repair cap is 2 rounds — on a third failure of the same criterion, stop and raise to your spawner (the user at the root, your parent when nested): at that point the evidence points at a wrong criterion or a wrong plan, not at code that needs another attempt.

## Hard rules

- **Never edit the acceptance criteria or any frozen section.** The contract's integrity is the point; verify runs a tamper check against the gate-1 state and an unbannered edit is its headline finding.
- **Never write implementation before its spec exists and fails.** Test-after converges on testing what you built instead of what was promised.
- **Never mark a task DONE with red or skipped specs.** DEVIATED-DONE still requires green — it flags *how*, not *whether*.
- **Never build unplanned scope, however good the idea.** Route it. The backlog is where good ideas wait their turn; the diff is not.
- **Never reference the plan document from shipped code.** Not in a comment, a commit message, a spec name, or a TODO. The plan lives outside the repository, so every such reference is a dangling pointer the next agent will trust.
