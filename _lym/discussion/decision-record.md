## Goal

```markdown
## Seen (orch, plan-split-bench, 2026-11-01)

In the `plan-split-bench` replays, Parts-Write fixed export names in `parts.md` that the repository's PATTERN entries forbid, and the part writers, who may not change `parts.md`, could not correct them:
- `internal/heddlecli#NewCommand`, where `PATTERN-cli-cobra` gives every CLI package `Command()`;
- names spelling `warp` in `heddleengine`, `fabricengine` and `cmd/lym`, which `PATTERN-fabric-vocabulary` bars and `validate-plan --part` refuses as `fabric-vocabulary-name`;
- two members that keep their signatures listed as exports, which `part-export-missing` cannot accept.

Each needed a hand edit of `parts.md` before the parts could join.
Parts-Write's prompt (`loom-template-parts.md`) carries the PATTERN directive, but nothing has it read the PATTERN entries that govern the names it fixes, and its own gate does not run the name checks the part gate runs.

## Goal (operator, 2026-11-01)

Tune the instructions of the plan-split writers, Parts-Write, the part writers and the joining chair, test them, and tune again, until a split plan joins and passes its gates with no hand edit.
Start from the `plan-split-bench` report's findings, which this task reads first.

## Changes to try first

- Parts-Write reads the PATTERN entries for every package a part exports from before it names an export, as the Plan writer does in its longer session.
- Parts-Write's gate runs the name checks `validate-plan --part` runs on export names, `fabric-vocabulary-name` included, and refuses an export naming an existing member whose signature the part keeps, so such a defect stops at Parts-Write, where it can still be fixed.
- Whatever else the bench report names as a cause of a hand edit or a failed join.

## Loop and cap

- Inputs: the two replays `plan-split-bench` used (`warp-visibility` and `run-verbs-shared`), at the same base commits.
- One round: tune the instructions, then replay Parts-Write and Parts-Plan-Write on both inputs with `plan_split` on, and record per input whether the parts joined, which gates refused, and every hand edit needed.
- At most 3 rounds, so at most 6 split writings; one Plan-Review round per input on the last round's plans only, 2 in all.
- The loop ends early at the first round where both inputs join and pass their gates with no hand edit.
- The orch creates and removes the scratch worktrees on the writer's request, since `lym fabric add` is refused from a task session.
- The instruction changes land in the stencils and gates; a report in `docs/research/` gives each round's results and the cost.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-02): Land round 1 without replay; replays move to plan-split-replay

Decision: Stencils live only in the hub-wide Bolt copy and production takes only `main`, so no replay can run this branch's lym and stencils before they land.
Split the loop at the landing: this run ends after round 1's tuning, with no replay and no scratch worktree.
Land the round-1 instruction and gate changes as they stand, once the gate passes.
Write the `docs/research/` report with the round-1 changes and their reasons, and mark the replay results as pending a follow-up task.
The follow-up task `plan-split-replay` runs the replays on the installed lym after this lands, and tunes further within the cap left over.

Rationale: Stencils are read only from the hub-wide Bolt copy and production deploys only main, so no replay can run this branch's lym and stencils before it lands.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - export-delete-resign-refusal-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
