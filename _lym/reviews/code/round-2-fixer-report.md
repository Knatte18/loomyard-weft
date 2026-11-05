# Round 2 fixer-report

## Changes

- **F1** (fixed as proposed, commit `2ab508b5`): `sketchRefusedNotice` in `internal/loomcli/wiring.go` words the first way forward as `lym board set-deps '{"slug":"<dependent>","depends_on":[...]}'` for each entry the refusal names, keeping its other dependencies. The sketch and raddle upserts are unchanged, and so is `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch`.
- **F2** (fixed as proposed, commit `3a7ff329`): the package doc in `internal/battencli/cli.go` now says the start guard judges the start as a fresh one first and the start removes the run once its owner check has passed (`claimStart`), so a refused start leaves the run in place.
- **F3** (fixed as proposed, commit `ddea6040`): `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` in `internal/battencli/dobbyreading_test.go` gains the row `ADoneRunOfARaddleEntryWhoseChildRecipeDoesNotReadHoldsTheStart`. A `childRecipeUnreadable` field makes the child-recipe record a directory, so reading it fails, and the row wants the start refused.

`lym gate test ./internal/battencli ./internal/loomcli` passed (tier0, 0 failed).

## Disputed

None.

## Deferred

None.

## Self-check

Forks run: fix-code-correctness, fix-consistency, fix-proposal, fix-affected-set. All four returned a report, none returned a finding, and the lens reports are saved beside this file as `round-2-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
