# Support log — fan-prompt-tuning

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry, the related entries `plan-writer-finding-classes`, `review-inline-code` and `loom-model-key-names`, both 2026-10-16 bench reports with their `setup/` folders, `internal/burlerengine` (config, profile, prompt, cluster audit, review table, embedded `burler.yaml`), the loom config template and its review-model resolution, the shed recipe's Burler and Bouncer wiring, `internal/twilldirective` with its stencils, and the burler and twill stencils.
Findings that shaped the design: the fan machinery, per-segment fan keys and the Bouncer's lens exclusions already exist; the fan's prompt text is Go prose in `clusterRulesBlock`; the fixer seat authorizes no forks; `twilldirective` is written for a writer, not a fixer; the hub's `burler.yaml` is seed-only, so new lens names reach it only through per-name fallback.

### Turn 2 — operator decision through `ly:orch`

`ly:orch` relayed an operator decision: the fan key and the fixer-mode key of each perch take one value or a per-round list, in the form of the per-segment model keys; the hub's standard setup is fan and twill fixer in round 1, solo and single from round 2; the orch sets the hub config after landing.
Recorded in D1 and built in D2; it forced D3 (retire `fan_review`) and D10 (exclusions per round).

### Turn 3 — lens pairing (moved from D8)

The reviewer's `interactions` looks across the whole artifact, `fix-consistency` only across what the fix touched; `bound-breaker` attacks the original claims, `fix-claims` the new ones; Plan's `affected-set` finds a partial set and `fix-affected-set` checks the fix closed it whole.

### Turn 4 — lens review

All six forks returned.

- premises, MEDIUM, D4 board-entry line ("fasit files" carries no board entry in production): fixed, variant D's `lyx board get` line stays.
- premises, LOW, new-fan-name reason false for this hub (its `burler.yaml` has no `discussion`/`plan` fan): fixed, reason is now keeping the dormant fans' meaning.
- premises, LOW, gate re-prompts of a twill fixer: fixed in D7's one-fork-pass rule and a plan-writer note.
- goal-scope, MEDIUM, `card-interactions` has no Plan evidence: fixed, lens dropped, D6 has three lenses.
- goal-scope, LOW, D6 rationale miscounts classes: fixed with the lens drop.
- goal-scope, LOW, `fan_review` retirement not marked as a self-made pick dropping the hub value: fixed in Auto-mode assumptions.
- goal-scope, LOW, fan-key defaults versus "builds the keys and their defaults": fixed, D2 says which keys the "default single" covers.
- goal-scope, LOW, Proposal E and `review-inline-code` silently gone: fixed, named in Out.
- board-fit, MEDIUM, `loom-model-key-names` clash: fixed, named in Open risks.
- board-fit, MEDIUM, `codex-engine` lists the fixer as fork-free: fixed, named in Open risks.
- board-fit, LOW, after-measure owner: fixed, Open risks points at `plan-writer-finding-classes`.
- board-fit, LOW, `orchestrator-survey-ideas` overlap: rejected, information only, no conflict.
- pattern-fit, MEDIUM, refusal-spec rows naming `fan_review` and new refusals: fixed in D2, D3, D9 and Scope.
- pattern-fit, MEDIUM, acceptance criteria pin static prose: fixed, criteria reworded to what Go fills.
- pattern-fit, LOW, `PATTERN-tolerant-records` misapplied to YAML: fixed, citation dropped.
- pattern-fit, LOW, Go prose is a current stencil-ownership departure: fixed in D4's rationale.
- leanness, MEDIUM, rejected alternatives inside D2 and D7: fixed, both bullets cut.
- leanness, LOW, bench narrative in rationales: fixed, trimmed to pointers.
- leanness, LOW, D8 pairing paragraph: fixed, moved to turn 3 with an anchor clause kept in D8.
- leanness, LOW, D1 restates later decisions: fixed, D1 points at them.
- leanness, LOW, Auto-mode assumptions repeat the ledger: fixed, now pointers per decision.
- leanness, LOW, non-constraints in Constraints: fixed, cut.
- leanness, LOW, D10 rationale undercuts it: fixed, leads with the per-row `ClusterExcludes` it breaks.
- leanness, LOW, empty turn 3: fixed, removed.
- leanness, note, Docs bullet in Scope is padding: rejected, it names the doc files the plan writer must list.
- scenarios, MEDIUM, a fork violation's end state: fixed, D9 stops the run with a named way forward and spec row.
- scenarios, MEDIUM, skipped self-check invisible: fixed, usage record counts forks, criterion covers zero.
- scenarios, MEDIUM, resume and relaunch of a twill fixer: fixed in D7.
- scenarios, MEDIUM, `fan_review` in an unmigrated layer: fixed in D3.
- scenarios, LOW, empty list undefined: fixed, refused in D2.
- scenarios, LOW, hub config change mid-run: fixed, lists resolved at run start.
- scenarios, LOW, fork-less engine refused late: rejected, the same spawn-time refusal a fanned reviewer has today; moving the check earlier is not this task's.

## Rejected alternatives

- **A `single`/`twill` enum key per perch plus a separate fixer lens key.** Two keys that must agree; a fan-name key selects the mode and names the lenses in one, as `discussion_lenses` does for the writer.
- **Keeping `fan_review` with a per-round meaning (the entry for round N applies when round N is fanned).** Two lists that must line up round by round; the segment's own per-round reviewer list already expresses it.
- **Rewriting the dormant `discussion` and `plan` fans in place.** Changes the meaning of names a profile may already use, and a hub `burler.yaml` defining those names would shadow the new texts.
- **A `card-interactions` lens for Plan-Review.** No Plan late finding shows two cards contradicting each other.
- **Rendering the fixer self-check through `internal/twilldirective`.** Its boilerplate and parts are a writer's (self-check passed, relaunch, writer fork); bending it for a fixer would make it less host-neutral, and the fixer's dispositions already live in the fix step.
- **A reviewer-level consequence check of each proposed fix.** Adds work to every reviewer and fork; `fix-proposal` checks only the proposals actually applied, at the fixer's cheaper context.
- **Failing the fix half on a fork-count mismatch, as the review half does.** It would redo a finished fix for a missed quality step.
- **A fixer key for Webster-Review.** The operator judged twill probably not needed for code, and nothing measured it there.
- **Keeping the fan prompt in Go and adding the bench text as a stencil override, as the bench did.** Leaves two conflicting fork rule sets in one prompt, the bench's named confound.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): how is the fixer mode selected? A fan-name key per perch, empty for `single` (D2). Rationale above in rejected alternatives.
- Q2 (auto-pick): what happens to `fan_review` under a per-round fan? Retired through `MigrateConfig` (D3).
- Q3 (auto-pick): new fan names or rewrite existing ones? New names (D5, D6, D8).
- Q4 (auto-pick): which Plan-Review lenses? `affected-set`, `guard-reach`, `way-forward`, one per Plan late-BLOCKING class (D6).
- Q5 (auto-pick): where is a reviewer's own flawed proposal caught? The `fix-proposal` fixer lens (D8).
- Q6 (auto-pick): which fixer lenses per perch? Bench's three plus `fix-proposal` for Discussion; `fix-way-forward` swapped for `fix-affected-set` on Plan (D8).
- Q7 (auto-pick): fixer fork audit strictness? Per-fork violations stop the run with a way forward, count mismatch warns and the usage record counts forks (D9).
- Q8 (auto-pick): does the twill fixer reuse `twilldirective`? No, a burler stencil part (D7).
- Q9 (auto-pick): Webster and Merge fixer keys? None (Scope).
- Q10 (auto-pick): should the judge's lens exclusions follow the per-round fan? Yes, asked only when the next round runs the same fan (D10).
- Q11 (auto-pick): template defaults? All four keys `""`; the hub's standard setup is the orch's after landing (D2, operator).
- Q12 (resolved, operator): per-round fan and fixer keys (D1, through `ly:orch`).
