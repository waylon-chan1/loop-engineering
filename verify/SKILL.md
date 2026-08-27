---
name: verify
description: Adjudicate a built lifecycle task against its frozen acceptance criteria with fresh eyes — re-run every spec yourself, exercise manual criteria against the running app, check the criteria weren't tampered with, and issue PASS/FAIL/BLOCKED/USER per criterion without fixing anything. Use when a lifecycle plan doc has Status built, after the diff audit.
---

# verify — fresh-eyes adjudication against the contract

Shared contract: `~/.claude/skills/lifecycle/LIFECYCLE.md`. Resolve the adapter first.

Verify exists because build cannot grade itself: the context that wrote the code will pattern-match its own claims and mark criteria done from memory. So verify runs **fresh** — a new session or a subagent that has not seen build's transcript — and re-derives every piece of evidence. A pasted transcript is not evidence; a fresh run is. Verify judges; it never repairs. The moment the verifier starts fixing what it found, it becomes the implementer and the grade is void.

## 0. Preconditions

- Plan doc Status must be `built` (run after the diff audit, since that mutates the diff — adjudicate what will actually ship). Wrong status → name the correct command and stop.
- Fresh context confirmed: if you implemented this task in this session, spawn a fresh subagent for the adjudication or instruct the user to run verify in a new session. Do not proceed as your own grader.
- `{workspace_bootstrap}` if needed.

## 1. Tamper check

Diff the `## Acceptance criteria` section against its state at the gate-1 approval commit (the doc's git history). Any change without a dated, user-approved supersession banner is a **headline finding** — report it first, before any verdicts, and treat affected criteria as unadjudicable until the user rules.

## 2. Adjudicate, criterion by criterion, in order

- **`[spec]`** — run the relevant `Verify:` commands yourself, then the full `{ci_command}` (or `{test_command}` + `{lint_command}`). Never accept a claim of green. For the core criterion, run a **mutation check**: stub or revert the key behavior change and confirm its spec goes red — a spec that can't fail guards nothing.
- **`[manual]`** — boot `{run_server}`; put the system into the needed state with `{mint_test_state}` (cold start via `{manual_probe}` if the repo has no minting tool); exercise the flow through `{manual_probe}`; observe the criterion with your own eyes. Record the exact requests/commands and observed responses into `## Verification evidence` — reproducible, so a skeptic can replay them.
- **`[user]`** — verdict is **USER** with the specific question the human must answer (what to look at, what "good" looks like). Never PASS on the human's behalf; a green-looking report with a human-owed item buried in it is this skill's canonical failure.

**Regression:** the full suite already ran above; additionally spot-check the plan's `## Failure Modes` table — each row's Test must exist and pass — and probe the nearest untouched behavior adjacent to the change.

## 3. Verdicts

**PASS / FAIL / BLOCKED / USER** per criterion.

- PASS requires reproducible evidence recorded in `## Verification evidence`: exact command + observed output. **No evidence = FAIL, not PASS.** Uncertainty resolves downward.
- BLOCKED names precisely what's missing (environment, credentials, data) and what would unblock it.
- Any FAIL or USER verdict is the **first line of the report**, never buried below the passes.

## 4. Route the outcome

- All PASS (+ USER items surfaced): Status → `verified-local`. Next command: open the PR; then the review stage (`{review_skill}` if the repo has one).
- Any FAIL: Status stays `built`. Hand the failing criteria back to build with the evidence attached. Verify never fixes code — not even the one-liner.

## Hard rules

- **Never adjudicate from build's claims, logs, or memory.** Re-derive or it didn't happen.
- **Never fix what you find.** Grading and repairing in one context recreates the self-grading this skill exists to break.
- **Never mark USER items PASS**, and never bury them — surface each with its specific question.
- **Never skip the tamper check.** It is the only enforcement the criteria freeze has.

## Report

FAIL/USER items first. Then the criterion table: number | statement (abbreviated) | method | verdict | evidence pointer. Then regression result, tamper-check result, Status change, and the exact next command.
