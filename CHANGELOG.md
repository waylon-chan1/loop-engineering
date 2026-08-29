# Changelog

Engine changes only — what changed in the process, and what motivated it. Newest first.

Two rules keep this file honest and publishable: an entry names the rule that changed and the *class* of evidence behind it, never the private task that surfaced it; and repository-specific facts never appear here, because they belong in an adapter, which is not part of this repository.

## 2026-08-28

- **Split the engine from its memory.** Real adapters moved to a private store and are symlinked back in, so this repository can be public while carrying no repository facts at all. Added a README, an MIT license, an example adapter, and a whitelist `.gitignore` — `~/.claude/skills` is where every user-level skill lands, so a file now joins the public engine deliberately rather than by being in the directory.
- **Tiered work by complexity, not size.** `plan-loop` classifies every task as patch, feature, or deep at Stage 0. Patch collapses to one combined gate; deep requires alternatives and a failure-mode registry. A two-file race-condition fix is deep; a twenty-file mechanical rename is not.
- **Tests split into scaffolding and contract.** Tests are unlimited during build, but only those tracing to a criterion, a failure mode, a regression, or a boundary contract survive into the PR. Prompted by a first live run that shipped 2,241 test lines against roughly 830 source lines.
- **Added the CHANGED verdict.** A criterion satisfied by different means than the plan described, backed by a logged deviation, is now distinguishable from both a failure and a silent deviation. Verify is conservative with PASS and generous with CHANGED: the goal is what was promised, not the route.
- **Findings must quote their evidence.** A FAIL cites the failing command and its output, or file:line and verbatim text. A failure that cannot be quoted is a suspicion, and a suspicion may not trigger the repair loop.
- **Added the planning pushback kit.** Premise challenge, claimed-limitations-need-evidence, and a ban on non-committal phrasing — balanced by never re-arguing a decision the approver has already made.
- **Added decision briefs and the outside voice.** Anything surfaced at a gate carries stakes, a recommendation, and trade-offs, classified as mechanical, taste, or challenge. Feature and deep tiers additionally get a cold-read subagent whose only charter is to find how the plan fails.
- **Added the observations flight recorder.** Any agent may append a dated, factual line to the plan document the moment it hits friction. Facts live there; only retro turns them into rules, and only when independently confirmed.
- **Lessons bucket by class and promote by spread.** Adjacency is judged by a controlled vocabulary rather than wording, `Times Seen` counts distinct tasks rather than mentions, and a lesson confirmed in one repository may only reach that repository's adapter. Two repositories are required before anything may change the engine — which is what keeps the engine generic by construction rather than by discipline.

The tier rubric's structure, the pushback kit, decision briefs, the outside voice, the CHANGED verdict, the regression rule, and the scaffolding-vs-contract policy were scavenged from [gstack](https://github.com/garrytan/gstack).

## 2026-08-27 — first version

- **The engine:** `plan-loop`, `build`, `verify`, and `retro`, over a shared contract that owns the artifact, the status machine, the gates, and the adapter keys. A skill that contradicts the contract loses.
- **Chaining.** Each stage spawns the next, so an approved task runs to a merge-ready PR unattended. Bounded by one successor per skill, a two-round repair cap, a twelve-hop ceiling, and hard stops at both gates, at any contradiction of the approved plan, and at merge.
- **Ownership.** Whoever spawns an agent owns it and answers its questions. Every agent carries the root approved plan, so it checks itself against a real contract instead of inferring intent, and the user is reached only when something contradicts that contract.
- **Cross-repository work.** Reading any repository is free; writing to one must be named in the plan and approved at gate 2. Each companion repository gets a complete nested lifecycle of its own, bounded to one level of recursion with no repository entered twice in a lineage.
- **Plans left the product repository.** The working copy lives in a git-backed plan store — a plan is process, not deliverable — and the PR description became the published copy that survives after merge.
- **Verification is fresh by construction.** Verify is always a newly spawned agent that never saw the build, and re-verification after a repair covers every criterion, because a fix that breaks a neighbouring criterion is exactly what an unattended loop would otherwise carry to the PR.
- **Reports are plain English.** Point first, concrete nouns, failures before passes, and none of the system's own vocabulary.
- **Autonomous improvement, with a boundary.** Retro applies qualifying graduations itself and reports after the fact, but may never autonomously weaken the gates, the criteria freeze, the qualification bar, or the boundary itself. A self-improving system's most likely catastrophic improvement is optimizing away its own oversight.
