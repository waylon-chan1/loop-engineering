---
name: verify
description: Adjudicate a built lifecycle task against its frozen acceptance criteria with fresh eyes — re-run every spec yourself, exercise manual criteria against the running app, check the criteria weren't tampered with, and issue PASS/CHANGED/FAIL/BLOCKED/USER per criterion without fixing anything. Use when a lifecycle plan doc has Status built, after the diff audit.
---

# verify — fresh-eyes adjudication against the contract

Shared contract: `~/.claude/skills/lifecycle/LIFECYCLE.md`. Resolve the adapter first.

Verify exists because build cannot grade itself: the context that wrote the code will pattern-match its own claims and mark criteria done from memory. So verify runs **fresh** — a new session or a subagent that has not seen build's transcript — and re-derives every piece of evidence. A pasted transcript is not evidence; a fresh run is. Verify judges; it never repairs. The moment the verifier starts fixing what it found, it becomes the implementer and the grade is void.

## 0. Preconditions

- Plan doc Status must be `built` (run after the diff audit, since that mutates the diff — adjudicate what will actually ship). Wrong status → name the correct command and stop.
- Fresh context, obtained without the user: if you implemented this task, or watched the repair that produced this state, **spawn a fresh subagent to adjudicate and let it report** — never ask the user to re-run verify in a new session. Freshness is the system's job to arrange, not a chore to hand to a human; a lifecycle that pauses for someone to open a terminal isn't autonomous. (In a chained run this holds automatically — verify always arrives newly spawned.)
- Read `## Chain log` to learn which repair round this is, if any. **Every round gets a genuinely new verifier**: not the agent that issued the previous verdict, and not one that saw the repair being made. Repeat verification is spawned automatically by the repair loop and needs no human to start it.
- **Re-verify the repair's blast radius, not the whole contract** (LIFECYCLE.md § Re-verification scope). Run every `[spec]` criterion regardless — that is one command and it is what keeps the narrowing safe. Re-exercise only the `[manual]`/`[user]` criteria the change could have moved: those served by a build-list task whose `Files:` this round's diff touched, plus any whose evidence runs through code the diff changed. Compute that from the diff and the build list; if it can't be computed — no `Files:` lines, shared or global code, a config or dependency change — the radius is everything and you re-verify in full.
- **Carry the rest forward, visibly.** A criterion outside the radius keeps its verdict, marked in `## Verification evidence` as carried from the round that derived it. Only a *previous verify agent's* recorded evidence may be carried, never build's claims, and only while its code stays untouched — a later diff reaching into it makes it re-derivable again.
- `{workspace_bootstrap}` if needed.

## 1. Tamper check

Diff the `## Acceptance criteria` section against its state at the gate-1 approval commit **in the plan store's git history** (`{plan_store}`). Any change without a dated, user-approved supersession banner is a **headline finding** — report it first, before any verdicts, and treat affected criteria as unadjudicable until the user rules.

## 2. Adjudicate, criterion by criterion, in order

- **`[spec]`** — run the relevant `Verify:` commands yourself, then the full `{ci_command}` (or `{test_command}` + `{lint_command}`). Never accept a claim of green. For the core criterion, run a **mutation check**: stub or revert the key behavior change and confirm its spec goes red — a spec that can't fail guards nothing.
- **`[manual]`** — boot `{run_server}`; put the system into the needed state with `{mint_test_state}` (cold start via `{manual_probe}` if the repo has no minting tool); exercise the flow through `{manual_probe}`; observe the criterion with your own eyes. Record the exact requests/commands and observed responses into `## Verification evidence` — reproducible, so a skeptic can replay them.
- **`[user]`** — verdict is **USER** with the specific question the human must answer (what to look at, what "good" looks like). Never PASS on the human's behalf; a green-looking report with a human-owed item buried in it is this skill's canonical failure.

**Cross-repo criteria:** when the plan has `## Companion repos`, a criterion that spans repositories is discharged at the **boundary** — run the companion repo's relevant tests yourself (its adapter names the commands) and exercise the contract both sides depend on, rather than accepting either side's claim about the other. A nested lifecycle's own verify report is *its* evidence, not yours: the criterion cannot PASS until that inner lifecycle is merge-ready, and you confirm that from the forge, not from its summary. Confirm any dependency you were told already shipped by querying the forge, not by finding a file locally. State the deploy order's consequence explicitly: if only one side ships first, does the system stay correct? An unanswered version of that question is a FAIL, not a footnote.

**Probe at the boundary your code owns.** To show that our code survives a dependency's failure, find where that dependency's own guarantees *end* — by reading its source — and inject there. A mock placed by guesswork tests the guess: mutating a library's internals usually proves only that the library handles its own errors, which is never what the criterion asked. When a criterion says "X still works when Y fails", the probe belongs at our call site, and the strongest version needs no stubbing at all.

**Regression:** the full suite already ran above; additionally spot-check the plan's `## Failure Modes` table — each row's Test must exist and pass — and probe the nearest untouched behavior adjacent to the change.

## 2b. Scope-drift check

Compare the diff's changed-file list against the build list's `Files:` lines (plus `{regen_artifacts}` outputs). Files changed that no task names are **"while-I-was-in-there" findings** — report each; unplanned scope is a finding even when the code is good.

## 2c. Second opinion (deep tier)

Deep-tier verdicts get one adversarial re-read from a different model family — `{outside_voice}`, per LIFECYCLE.md § The outside voice. Send it the plan document, the diff, and your draft verdicts with their evidence. Charter narrows to: *which of these verdicts is wrong, and what evidence shows it.*

**A verdict dispute is settled by re-running the command, never by argument.** This is the one stage where the outside voice's disagreement has ground truth underneath it — a spec is red or it is not — so resolve it there and record the run that resolved it. A dispute that survives the re-run is a real finding and reaches the report as one.

It may not repair anything either; run it read-only, which makes this skill's no-fixing rule mechanical rather than a promise. If it could not be run, the report says the verdicts had a single adjudicator instead of implying two.

## 3. Verdicts

**PASS / CHANGED / FAIL / BLOCKED / USER** per criterion.

- PASS requires reproducible evidence recorded in `## Verification evidence`: exact command + observed output. **No evidence = FAIL, not PASS.** Uncertainty resolves downward.
- **CHANGED** = the criterion is satisfied, but by different means than the plan described — and a `## Deviations` entry covers the difference. Same evidence bar as PASS. An equivalent-but-different implementation with a logged deviation is CHANGED, not FAIL and not tampering; the same difference with **no** logged deviation is a silent-deviation finding.
- Postures: **be conservative with PASS** — a file being touched is not evidence; code that *handles* a deliverable is not the deliverable. **Be generous with CHANGED** — the goal is what was promised, not the route. **Be honest with BLOCKED** — name precisely what's missing and the exact manual check that would discharge it; "I don't want to check" is not BLOCKED.
- **Evidence gate on FAIL:** a FAIL must quote what motivates it — the failing command and its output, or file:line and the verbatim text. A failure you cannot quote is a suspicion: it goes in a report appendix, never a verdict, and is not permitted to trigger the repair loop.
- Any FAIL or USER verdict is the **first line of the report**, never buried below the passes.

## 4. Route the outcome (the chain's branch point)

Unless invoked `no-chain`, append a `## Chain log` line and spawn **one** successor per the outcome:

- **All PASS/CHANGED, no USER items** → Status `verified-local`; spawn an agent to run the **review-loop skill** (it opens the PR, publishes the reviewer-facing rendering, and drives comment adjudication to merge-ready; merging is the human's).
- **Only BLOCKED verdicts remain** (every criterion PASS/CHANGED except items needing environment/credentials/data you cannot obtain) → Status stays `built`; no successor. Report each BLOCKED item with the exact manual check that would discharge it — a PR must not open with unadjudicated criteria.
- **Any FAIL** (within the 2-round repair cap) → Status stays `built`; spawn a build agent in repair mode with the failing criteria and their evidence attached. Verify never fixes code — not even the one-liner; grading and repairing in one context is the failure this skill exists to prevent.
- **FAIL past the repair cap, or any USER verdict, or a tamper finding** → no successor. Stop and report to your spawner — the user at the root, your parent agent when nested (a USER verdict always ends up with the human, since only they can discharge it). Three failures against one criterion is evidence about the criterion, not about the code; a USER item is by definition not yours to discharge.

## Hard rules

- **Never adjudicate from build's claims, logs, or memory.** Re-derive or it didn't happen. The single exception is a criterion carried forward from outside the blast radius, which rests on a previous *verifier's* recorded evidence plus its code being untouched — never on anything build reported.
- **Never ask the user to run a verification for you.** If you need a clean context, spawn one. The only things that stop this chain are a USER verdict, a tamper finding, or the repair cap.
- **Never fix what you find.** Grading and repairing in one context recreates the self-grading this skill exists to break.
- **Never mark USER items PASS**, and never bury them — surface each with its specific question.
- **Never skip the tamper check.** It is the only enforcement the criteria freeze has.

**Observations:** friction during adjudication — a criterion that was hard to interpret, evidence that was hard to obtain, tooling gaps — gets one dated, stage-tagged, factual line in `## Observations`. Check once before reporting: anything observed, not yet logged? Zero is fine; skipping the check is not.

## Report

Plain English, short, point first (LIFECYCLE.md § How findings are reported).

Anything broken or needing you goes at the top, one line each: what failed, where, what it means. Then a compact table — criterion | what it checks | how checked | verdict | evidence. Then regression and tamper-check results in a line apiece, the second opinion's outcome when deep tier called for one — corroborated, conceded, or still disputed — the Status change, and the next command.

Write it for someone who has never read this contract: "criterion 3 fails — the API returns 200 with an empty body when it should return 422" rather than "criterion 3 FAIL, envelope conformance divergence." If everything passed, say so in one line and stop.
