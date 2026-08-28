---
name: verify
description: Adjudicate a built lifecycle task against its frozen acceptance criteria with fresh eyes — re-run every spec yourself, exercise manual criteria against the running app, check the criteria weren't tampered with, and issue PASS/FAIL/BLOCKED/USER per criterion without fixing anything. Use when a lifecycle plan doc has Status built, after the diff audit.
---

# verify — fresh-eyes adjudication against the contract

Shared contract: `~/.claude/skills/lifecycle/LIFECYCLE.md`. Resolve the adapter first.

Verify exists because build cannot grade itself: the context that wrote the code will pattern-match its own claims and mark criteria done from memory. So verify runs **fresh** — a new session or a subagent that has not seen build's transcript — and re-derives every piece of evidence. A pasted transcript is not evidence; a fresh run is. Verify judges; it never repairs. The moment the verifier starts fixing what it found, it becomes the implementer and the grade is void.

## 0. Preconditions

- Plan doc Status must be `built` (run after the diff audit, since that mutates the diff — adjudicate what will actually ship). Wrong status → name the correct command and stop.
- Fresh context confirmed: if you implemented this task in this session, spawn a fresh subagent for the adjudication or instruct the user to run verify in a new session. Do not proceed as your own grader. (In a chained run this holds automatically — verify always arrives as a newly spawned agent.)
- Read `## Chain log` to learn which repair round this is, if any.
- `{workspace_bootstrap}` if needed.

## 1. Tamper check

Diff the `## Acceptance criteria` section against its state at the gate-1 approval commit **in the plan store's git history** (`{plan_store}`). Any change without a dated, user-approved supersession banner is a **headline finding** — report it first, before any verdicts, and treat affected criteria as unadjudicable until the user rules.

## 2. Adjudicate, criterion by criterion, in order

- **`[spec]`** — run the relevant `Verify:` commands yourself, then the full `{ci_command}` (or `{test_command}` + `{lint_command}`). Never accept a claim of green. For the core criterion, run a **mutation check**: stub or revert the key behavior change and confirm its spec goes red — a spec that can't fail guards nothing.
- **`[manual]`** — boot `{run_server}`; put the system into the needed state with `{mint_test_state}` (cold start via `{manual_probe}` if the repo has no minting tool); exercise the flow through `{manual_probe}`; observe the criterion with your own eyes. Record the exact requests/commands and observed responses into `## Verification evidence` — reproducible, so a skeptic can replay them.
- **`[user]`** — verdict is **USER** with the specific question the human must answer (what to look at, what "good" looks like). Never PASS on the human's behalf; a green-looking report with a human-owed item buried in it is this skill's canonical failure.

**Cross-repo criteria:** when the plan has `## Companion repos`, a criterion that spans repositories is discharged at the **boundary** — run the companion repo's relevant tests yourself (its adapter names the commands) and exercise the contract both sides depend on, rather than accepting either side's claim about the other. A nested lifecycle's own verify report is *its* evidence, not yours: the criterion cannot PASS until that inner lifecycle is merge-ready, and you confirm that from the forge, not from its summary. Confirm any dependency you were told already shipped by querying the forge, not by finding a file locally. State the deploy order's consequence explicitly: if only one side ships first, does the system stay correct? An unanswered version of that question is a FAIL, not a footnote.

**Regression:** the full suite already ran above; additionally spot-check the plan's `## Failure Modes` table — each row's Test must exist and pass — and probe the nearest untouched behavior adjacent to the change.

## 3. Verdicts

**PASS / FAIL / BLOCKED / USER** per criterion.

- PASS requires reproducible evidence recorded in `## Verification evidence`: exact command + observed output. **No evidence = FAIL, not PASS.** Uncertainty resolves downward.
- BLOCKED names precisely what's missing (environment, credentials, data) and what would unblock it.
- Any FAIL or USER verdict is the **first line of the report**, never buried below the passes.

## 4. Route the outcome (the chain's branch point)

Unless invoked `no-chain`, append a `## Chain log` line and spawn **one** successor per the outcome:

- **All PASS, no USER items** → Status `verified-local`; spawn an agent to open the PR and run `{review_skill}`. **That agent writes the plan's reviewer-facing rendering into the PR description** (LIFECYCLE.md § The artifact) — criteria with verdicts and evidence, deviations, companion repos and deploy order, NOT-in-scope with routing — and confirms the plan document itself is absent from the diff. It stops when the PR is merge-ready; merging is the human's.
- **Any FAIL** (within the 2-round repair cap) → Status stays `built`; spawn a build agent in repair mode with the failing criteria and their evidence attached. Verify never fixes code — not even the one-liner; grading and repairing in one context is the failure this skill exists to prevent.
- **FAIL past the repair cap, or any USER verdict, or a tamper finding** → no successor. Stop and report to the user. Three failures against one criterion is evidence about the criterion, not about the code; a USER item is by definition not yours to discharge.

## Hard rules

- **Never adjudicate from build's claims, logs, or memory.** Re-derive or it didn't happen.
- **Never fix what you find.** Grading and repairing in one context recreates the self-grading this skill exists to break.
- **Never mark USER items PASS**, and never bury them — surface each with its specific question.
- **Never skip the tamper check.** It is the only enforcement the criteria freeze has.

## Report

Plain English, short, point first (LIFECYCLE.md § How findings are reported).

Anything broken or needing you goes at the top, one line each: what failed, where, what it means. Then a compact table — criterion | what it checks | how checked | verdict | evidence. Then regression and tamper-check results in a line apiece, the Status change, and the next command.

Write it for someone who has never read this contract: "criterion 3 fails — the API returns 200 with an empty body when it should return 422" rather than "criterion 3 FAIL, envelope conformance divergence." If everything passed, say so in one line and stop.
