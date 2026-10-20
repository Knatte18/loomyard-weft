# lenses: twill-plan runs every review-plan lens

The template `twill-plan` fan in `internal/lenses/template.yaml` gains `affected-set`, `guard-reach` and `way-forward`, the three lenses the `review-plan` fan judges a Plan by.
The fan now reads `affected-set`, `guard-reach`, `way-forward`, `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
The Plan writer thereby runs the procedures its reviewer will use and finds its own holes before the review, as the Discussion writer already does.
The audit `twill-writer-quality-audit` found that procedure lenses halved round-1 Plan findings, while the Discussion writer, whose lenses were emphasis lenses, cut only about 20%.

The Discussion side needs no change: the template `twill-discussion` fan already holds every `review-discussion` lens.

## Design decisions

- The reviewer's lenses come first in the fan, matching the order of `twill-discussion`.
- The five existing writer lenses stay. None of the three reviewer lenses carries their checks: `forced-edits` checks a `Create` handle's package clause, the quarry glyphs and table registrations, and `branch-coverage` and `row-reachability` check test rows, which `guard-reach` does not.
- Lens prose is untouched, so a reviewer's lens and the writer's copy read the same.
- The fan grows from five forks to eight per Plan-Write, which is under `MaxFanLenses`.
- Webster, Merge and Darn are out of scope: their writers are implementers with no writer lens key, so giving them one is code, not a fan change.

## Tests

`TestConfigTemplate_DecodesAndIsSelfConsistent` in `internal/lenses/config_test.go` updates the `twill-plan` count and order, and gains a check that each writer fan (`twill-discussion`, `twill-plan`) contains every lens of its reviewer fan (`review-discussion`, `review-plan`).

## To check or run by hand

- Reconcile keeps a file's fan lists unchanged, so the new fan reaches a hub only through a `lenses.yaml` with no `twill-plan` entry.
  The hub's `lenses.yaml` on Bolt holds `twill-plan` verbatim with five lenses and keeps them until its entry is edited or deleted; that edit is outside this change.
- The extra forks cost more per Plan-Write. Whether that pays is decided by comparing round-1 Plan findings over the next runs against the audit baseline.
- `lyx gate test --tags integration ./...` was not run as part of writing this description.
