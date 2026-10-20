# Decision record — writer-reviewer-lens-align

## Goal

Each perch writer's twill lens fan runs the procedure lenses its reviewer's fan judges it by, so the writer finds its own holes before the review.

## Scope

- In scope: the template `twill-plan` fan in `internal/lenses/template.yaml`, and the test that pins the template's fans.
- Already done: the Discussion side, since the template `twill-discussion` holds every `review-discussion` lens.
- Out of scope: Webster, Merge and Darn, whose writers are implementers with no writer lens key; giving them one is code, not a fan change.
- Out of scope: lens prose; every lens stays as written, so a reviewer's lens and the writer's copy of it read the same.
- Out of scope: the hub's own `lenses.yaml` on Bolt, which holds `twill-plan` verbatim and so keeps its five lenses until its entry is deleted or edited; that edit is the operator's or the orch's, outside this worktree.

## Decisions

### D1: `twill-plan` gains the three `review-plan` lenses

Decision: the template `twill-plan` fan becomes `affected-set`, `guard-reach`, `way-forward`, `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
The reviewer's lenses come first, as in `twill-discussion`.

Rationale: `review-plan` holds `affected-set`, `guard-reach` and `way-forward`, none of which the Plan writer runs today.
The five writer lenses stay, since they are procedure lenses that already target the finding kinds that fell at Plan, and none of the three reviewer lenses carries their checks: `forced-edits` checks a `Create` handle's package clause, the quarry glyphs and table registrations, `branch-coverage` and `row-reachability` check test rows, which `guard-reach` does not.
The fan grows from five forks to eight, under `MaxFanLenses`.

## Constraints

- `PATTERN-test-economy`: the template-fan test gains no new test; its `twill-plan` count and order rows change.
- Reconcile keeps a file's fan lists unchanged, so the new fan reaches a hub only through a `lenses.yaml` that lacks a `twill-plan` entry.

## Auto-mode assumptions

- No operator is present; the fan order and the choice to keep all five writer lenses are self-picks, logged in the support log's question ledger.

## Open risks

- Eight forks cost more per Plan-Write than five; the operator's measurement of round-1 Plan findings decides whether the cost pays.

## Acceptance criteria

- The template `twill-plan` fan resolves to the eight lenses of D1, in that order.
- Every `review-plan` and `review-discussion` lens is in the matching template `twill-plan` or `twill-discussion` fan.

## Notes for the plan writer

- `TestTemplate` in `internal/lenses/config_test.go` holds the fan count and order tables to change.
