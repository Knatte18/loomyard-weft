# Plan-Review round 2 — fixer report

All three findings hold against the code; each is fixed.

## Changes

- **F1** (fixed as proposed) — `_lyx/plan/08-begin-batch-builds-pack.md`.
  Confirmed: `BeginBatch` runs `restampFingerprint` right after `ValidateDispatch` (`internal/websterengine/beginbatch.go`), and its `overviewFrameHash` goes through `planparser.OverviewWithoutCardIndex`, which fails on an overview with no frontmatter or no Card Index, so a garbage overview refuses `begin-batch` before `buildKickstartPack`.
  The `breakPlanAfterValidate` override now appends a Card Index line for card 3, which has no card file.
  The card states why `restampFingerprint` accepts it (frontmatter and `## Card Index` intact, the batch's card files still hash) and why `ParsePlan` refuses it (`parseCardFile` finds no file for card 3).
  `beginbatch.go` calls `ParsePlan` nowhere between the restamp and the render, so the fresh parse in `buildKickstartPack` is the first to fail.
- **F2** (fixed as proposed) — `_lyx/plan/08-begin-batch-builds-pack.md`.
  Confirmed: `ValidateDispatch` runs `planparser.ValidateFormat` (`internal/planglyph/planglyph.go`), `checkCardMissingField` requires `ImpactSummary:` on an `Edit` or `Delete` group (`internal/planparser/validate.go`), and `plankit.Write` renders it only when set.
  The fixture paragraph now says card 1 carries an `ImpactSummary`, so `ValidateDispatch` reports nothing blocking.
  This covers the draft-handle row's mixed `Create` + `Edit` card 1 as well, since the paragraph governs every row's card 1.
- **F3** (fixed as proposed) — `_lyx/plan/05-bind-trace.md`.
  Confirmed: `newRecordFixtureOver` (`internal/websterengine/recordbatch_test.go`) gives an empty plan directory and a one-card batch.
  The test paragraph now has each row write its plan with `plankit.Write`, as `writeRecordPlanDir` does, point `Deps.Geom.PlanDir`, `Deps.Plan`, `Deps.Batches` and batch 1's begun `Cards` at it, give the mixed row a two-card batch 1, re-stamp with `websterengine.RestampPlanBaseline`, and set `fakeIndex.delta` to the created symbols its handles match.

## Disputed

None.

## Deferred

None.
