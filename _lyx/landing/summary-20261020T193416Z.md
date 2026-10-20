# lenses: twill-plan runs the review-plan lenses

The Plan writer's twill fan now runs the procedures its reviewer judges it by, so the writer finds those holes before Plan-Review does.

## Change

- `internal/lenses/template.yaml`: the template `twill-plan` fan becomes `affected-set`, `guard-reach`, `way-forward`, `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
  The three `review-plan` lenses come first, as the `review-discussion` lenses do in `twill-discussion`.
  No lens prose changes, so the writer's copy of each reviewer lens reads the same as the reviewer's.
- `internal/lenses/config_test.go`: `TestConfigTemplate_DecodesAndIsSelfConsistent` pins the new count and order, and checks that each template writer fan (`twill-discussion`, `twill-plan`) holds every lens of its reviewer fan (`review-discussion`, `review-plan`).

The Discussion side needed no change: the template `twill-discussion` already holds every `review-discussion` lens.

## Design decisions

- All five existing writer lenses stay beside the three reviewer lenses, so the fan grows from five forks to eight, under `MaxFanLenses`.
  `forced-edits` overlaps `affected-set`, but it also checks a `Create` handle's package clause, the quarry glyphs and table registrations, which no `review-plan` lens covers.
  `branch-coverage` and `row-reachability` check test rows, which `guard-reach` does not.
- Webster, Merge and Darn are left alone: their writers are implementers and have no writer lens key, so aligning them would mean new code, not a fan change.

## Follow-up for the hub

Reconcile keeps a `lenses.yaml` fan list unchanged.
The hub's board `lenses.yaml` holds `twill-plan` with the old five lenses word for word, so this hub keeps running those five until that entry is deleted or edited.
That file is not part of this change.
