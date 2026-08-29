# Loop Engineering

A near-autonomous development lifecycle built out of [Claude Code](https://claude.com/claude-code) skills. It enforces one discipline: **concrete acceptance criteria are written and human-approved before any code exists**, implementation is test-first against exactly those criteria, and a fresh-eyes verifier — an agent that never saw the implementation — adjudicates the result against the frozen contract.

```
plan-loop A ─ gate 1 ─▶ plan-loop B ─ gate 2 ─▶ build ─▶ diff audit ─▶ verify ─▶ PR ─▶ review ─▶ merge ─▶ retro
  (criteria)              (build plan)           (TDD)                  (fresh eyes)                    (lessons)
```

An engineer prompting an agent starts from scratch every time. This is an attempt at the opposite: a lifecycle that compounds — each task's misses are attributed to the stage that should have caught them, and recurring ones graduate into the skills themselves.

## The ideas that do the work

**Criteria are frozen, and the freeze is checkable.** Approving the criteria is a commit in a git-backed plan store. Verify diffs the criteria section against that commit; an unbannered edit is its headline finding, reported before any verdict. The contract cannot be quietly bent to match what got built.

**Nothing grades its own work.** Verify always runs as a newly spawned agent that has not seen build's transcript. A pasted transcript is not evidence — the verifier re-runs every command itself, and PASS requires reproducible evidence recorded in the document. No evidence is a FAIL, not a PASS.

**The document is the orchestrator, not any session's memory.** A `Status:` line (`planning → criteria-approved → plan-approved → building → built → verified-local → merged → closed`) drives everything. Each skill refuses to run out of order and names the command the task actually needs. Any phase resumes in a fresh session with no context handed over.

**It chains itself, within stated bounds.** Each stage spawns the next, so an approved task runs to completion unattended. The bounds are what make that safe: at most one successor per skill, a 2-round verify→repair→re-verify cap, a 12-hop ceiling, and hard stops at both human gates, at any contradiction of the approved plan, at any verdict only a human can discharge, and at merge.

**Whoever spawns an agent owns it.** Subagents answer to their spawner, not to the user. Every agent also receives the root approved plan, so it holds itself to the contract a human actually signed rather than guessing at intent. The user is reached by escalation only, and only for the one thing a spawner cannot decide: something that contradicts that contract.

**Work is tiered by complexity, not size.** A two-file race-condition fix is `deep`; a twenty-file mechanical rename is not. `patch` collapses to one combined gate; `feature` adds a cold-read outside-voice review of the plan; `deep` requires alternatives and a failure-mode registry. Tier is confirmed at the first gate and then committed to.

**Tests split into scaffolding and contract.** During build, tests are unlimited — red-green stepping stones are free and drive correctness. At PR time only tests tracing to a criterion, a failure mode, a regression, or a boundary contract survive. (This rule exists because the first live run shipped 2,241 test lines against ~830 source lines.)

**It improves itself, and cannot improve away its own oversight.** Misses and catches both go to a lesson tracker; a lesson graduates into a skill only on repeated independent confirmation or one severe miss, as the smallest edit that would have prevented it, committed with the evidence cited. A lesson confirmed in one repository may only reach that repository's adapter — it takes two to touch the engine, which is what keeps the engine generic by construction. The hard boundary: **the system may never autonomously weaken its own human controls** — gates, the criteria freeze, the qualification bar, and the boundary itself change only with explicit approval.

## Layout

| Path | What it is |
|---|---|
| `lifecycle/LIFECYCLE.md` | The shared contract: artifact, status machine, gates, chaining and recursion bounds, deviation classes, adapter keys, the iteration procedure. If a skill contradicts it, it wins |
| `plan-loop/SKILL.md` | Stage A frames the task into criteria (gate 1); Stage B scopes the implementation (gate 2) |
| `build/SKILL.md` | Executes the approved plan test-first, logging every deviation; stops rather than improvising a redesign |
| `verify/SKILL.md` | Fresh-eyes adjudication: PASS / CHANGED / FAIL / BLOCKED / USER, with evidence |
| `retro/SKILL.md` | Harvests confirmed lessons after merge and graduates the recurring ones |
| `lifecycle/contexts.example/EXAMPLE.md` | A template adapter — the only place repository facts are allowed to live |

## Using it

```sh
git clone https://github.com/waylon-chan1/loop-engineering ~/.claude/skills
cp ~/.claude/skills/lifecycle/contexts.example/EXAMPLE.md \
   ~/.claude/skills/lifecycle/contexts/<your-repo>.md   # then fill it in
```

Then, in a repository: `/plan-loop <what you want built>`, approve the criteria, approve the build plan, and the rest runs on its own until it needs you.

The four skills contain **zero repository facts**. Commands, conventions, bootstrap steps, follow-up routing, and companion repositories all resolve through the adapter, so a new repository gets the full lifecycle by writing one file. A missing adapter key degrades that capability with a stated reason rather than blocking — and no skill invents tooling a key doesn't name.

The diff-audit and review stages are deliberately slots rather than implementations: point them at whatever your repository already uses, or let the generic descriptions apply.

## What is not here, on purpose

Real adapters, plan documents, and lesson rows are the *memory* of a specific team's codebase. They stay private. This repository is the engine — the process — and it is written so that the engine is genuinely all you need to start.

## Status

Honest state: in battle-testing. The pipeline has driven real work through gates, TDD, a diff audit, three verify rounds, two bounded repairs, and a repair-cap stop that correctly refused to keep trying. The self-improvement half — retro, the tracker, graduation — is implemented but has not yet run on a merged task, so treat its rules as designed rather than proven. Sharp edges are documented in the skills, not hidden.

Scavenged in part from [gstack](https://github.com/garrytan/gstack) (2026-08-28): the tier rubric's structure, the planning pushback kit, decision briefs, the outside voice, the CHANGED verdict, and the scaffolding-vs-contract test policy.

## License

MIT
