---
name: retro
description: Harvest lessons after a lifecycle task merges — mine the review adjudication tally, PR threads, and verify failures for independently-confirmed misses and catches, attribute each to the stage that should have caught it, record them in the tracker, and graduate recurring lessons into the skills themselves. Use when a lifecycle plan doc has Status merged, or to run the periodic lesson sweep.
---

# retro — close the loop

Shared contract: `~/.claude/skills/lifecycle/LIFECYCLE.md`, especially § Lessons and § Iterating on the lifecycle itself. Resolve the adapter first.

Retro is the mechanism that makes this a *loop*: it converts what actually happened on a finished task into durable pressure on the skills. Its discipline is selectivity — a tracker full of noise is worse than no tracker, because the planning skills read it and steer by it. The failure this skill guards against by design: learning from bogus review comments and hallucinating down the wrong path.

## 0. Preconditions

- Plan doc Status `merged` (or invoked in sweep mode, below). Wrong status → name the correct command and stop.
- A nested lifecycle (one with a `Parent:` header) is harvested **as its own task**, attributed to its own repo — its misses belong to the stage that made them there, not to the parent. Read the parent for the authorization context; don't merge the two records.
- **Sweep mode** (the autonomous entry point, run on a schedule): list plan docs in `{plan_store}` with Status `merged` — the store spans repos, so use each doc's `Repo:` header to work in the right one — run this skill per doc, then the dead-rule audit. Setting `closed` at the end of each doc (step 5a) is what makes the sweep idempotent — a doc is never harvested twice.
- Inputs to gather: the review skill's adjudication tally (finding → verdict → reason), the PR's thread history and commits, verify's report (FAILs, USER outcomes, tamper findings), and the plan doc's `## Deviations`, `## Observations` (the flight recorder every stage appended to), and any human overrides during the run.
- **Harvest is unconditional.** This step always runs in full; "the run felt clean" is not a reason to skip it. Zero qualified lessons is the explicit line "No durable lessons from this task" — an empty result, never a skipped step. If the local working copy is missing, the merged PR description holds the published plan — recover it with `gh pr view`.

## 1. Qualify candidates — the bar, in full

A candidate becomes a tracker row only if it is **all** of (LIFECYCLE.md § Lessons):

- generalizable, actionable, specific, durable — the class and the fix direction, mechanically applicable, still true after this PR;
- **independently confirmed** — one of: a review finding whose verdict was a real fix; a human comment that produced a commit; a verify FAIL; a plan-error deviation. **A dismissed bot comment never qualifies** — recording it trains the system on noise, which is exactly the wrong-path hallucination this bar prevents. At most, a *pattern of confident-wrong bot findings* is itself a lesson for the review stage.
- clean — never quote user data or secrets into a lesson.

Also qualify **catches** (`went-well`): a gate or rule that demonstrably stopped a defect or scope creep. Gaps and working parts must both be visible, and the dead-rule audit needs the positive signal.

## 2. Attribute

Attribute each miss to the **earliest stage that could have prevented it** — that stage's skill owns the fix:

| Symptom | Stage |
|---|---|
| Defect a criterion should have covered; missing/untestable criterion; wrong scope call | plan |
| Spec written to pass rather than to fail first; unplanned scope built; silent deviation | build |
| Evidence accepted without re-derivation; USER item buried; regression missed | verify |
| Bad fix made because a bot said so; real finding wrongly dismissed | review |
| Change shipped against the tally's own verdict with no new evidence (`caved-without-evidence` — a bypassed control: severe, graduates immediately) | review |
| Noise stored; real lesson missed | retro |

## 3. Record — bucket, then count

At `{tracker_location}`, per LIFECYCLE.md § Lessons:

- **Bucket by the controlled vocabulary, not by wording.** Same stage + same Class + same subject → the existing row; bump `Times Seen`. A candidate no class fits either gets an existing class stretched (say so) or — deliberately, reported as such — a new class.
- **Times Seen counts distinct tasks.** All observations of one friction within this task bump the counter by exactly one, however many agents hit it.
- Observations from `## Observations` follow the same bar as everything else: corroborated (verify FAIL, review fix, human commit, plan-error) → candidate; uncorroborated → dies with the task, unrecorded. That is the filter, not a loss.
- New rows carry: Lesson, Stage, Class, Outcome, Detail, Source, **Repo**, Task/PR, Times Seen = 1, Status `active`, Date.
- **Cap: ~10 active rows per stage.** At the cap, adding means merging or archiving first — pick the weakest-attested row and fold or retire it, noting where it went.
- Tracker unreachable → put the qualified lessons in the report verbatim and stop there; never block on the tracker, never retry in a loop.

## 4. Graduate — changing the system itself

Threshold: **Times Seen ≥ 3 (distinct tasks), or one severe miss** (shipped defect, bypassed gate) — severity always outranks the counter. Then, per LIFECYCLE.md § Iterating:

1. Decide the layer — and **the evidence's spread decides it, not judgment**: a lesson whose confirmations all come from one repo may only graduate into that repo's adapter (or conventions file), at any count. Only confirmation across ≥2 repos qualifies a lesson for engine text (the owning skill or LIFECYCLE.md). This is what keeps the engine plug-and-play: single-repo experience physically cannot reach it.
2. **Contradiction check.** Before applying, test the lesson against what already stands: does it contradict an existing skill rule, `{conventions_files}`, or `{invariants}`? A contradiction is not a tiebreaker in the lesson's favor — it means either the rule is wrong or the feedback was bad, and that call goes to the user with both sides' evidence. Bad feedback that slipped past the qualification bar gets caught here, before it can rewrite a rule that was right.
3. Make the **smallest edit that would have prevented the miss**. Skill edits happen between runs, never mid-run, and must cite the tracker rows demanding them — the same evidence rule the skills enforce on code.
4. Commit in `~/.claude/skills` (it is a git repo), message citing the tracker row.
5. Record it in both places, then mark the row `graduated`: a dated line in `CHANGELOG.md` at the engine repo root — which is **public**, so name the rule that changed and the class of evidence, never the private task that surfaced it — and a dated entry in the stage's Iteration Log with what happened → what changed → row link.

While the skills are user-level, graduation edits them directly — apply, commit, log, then **report what changed after the fact; don't ask permission first.** Once a skill has moved into a repo, graduation goes through that repo's normal PR flow.

**The autonomy boundary (LIFECYCLE.md § Iterating — it outranks the autonomy above):** never autonomously edit gate definitions or approval requirements, the criteria freeze, the lesson qualification bar, the deviation classes that route to the user, or the boundary itself. Those changes are *proposed* to the user with the evidence, and applied only on approval. A self-improving system must not be able to optimize away its own oversight.

## 5a. Close the task and end the chain

Set the plan doc's Status → `closed` and commit it **in the plan store**. `closed` is the terminal state; it marks the doc as harvested. Nothing about closing a task touches the product repo.

Retro spawns **no successor** — it is the end of the lifecycle. In sweep mode it iterates the remaining docs itself rather than chaining, so a backlog of merged tasks can never fan out into a swarm.

## 5b. Dead-rule audit (sweep mode only)

Any rule with no associated catch across many runs is a deletion candidate — propose the cut with the evidence of silence. Skills must not monotonically bloat; every rule is either earning catches or on notice.

## Hard rules

- **Never store a lesson from a dismissed finding.** The tracker steers future planning; feeding it noise steers the whole system wrong.
- **Never edit a skill mid-run or without citing tracker rows.** Unattributed edits destroy the evidence chain that makes iteration converge.
- **Never quote user data or secrets into a lesson.** The PII rule applies to lessons exactly as to logs.
- **Never write to another skill's learning store** (e.g. a reviewer-side database named in the adapter as off-limits).

## Report

Plain English, short, point first (LIFECYCLE.md § How findings are reported).

Lead with what you changed about the system and why — one line each, in the form "we kept doing X, so now the skill does Y." Then what you chose not to record and why, anything you're escalating instead of applying, and whether the tracker was reachable. Skip the ceremony; a reader should learn in ten seconds what the system got better at this week.
