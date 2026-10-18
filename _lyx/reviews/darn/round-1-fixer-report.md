## Changes

All four findings fixed as proposed.

- F1: `weeks_test.go` gains a row with two loom runs and one darn run in one week, asserting `Runs == 3` and `Recipes == {loom: 2, darn: 1}`.
- F2: the darn row of `TestComputeTiming` gains a batten record whose Run-Shed change names `Darn-Burler-Review` between the `Darn-Write` and `Darn-Bouncer` rows.
  The 30s counts under review, and the loom-table lookup would put it under write (write 210, review 150), so the unchanged 180/180 assertions fail with the defect in place.
- F3: the `runs.go` header says a full run is one whose records show a finished run of a selected recipe, loom or darn.
- F4: the `phaseOf` doc comment says "child row".

Gate: `lyx gate test ./internal/reportengine ./internal/reportcli` passes.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: `landings.go` `Landing` doc said "finished loom run"; fixed as proposed, now "finished run".
- fix-consistency, LOW: remaining "loom row" in the `phaseOf` doc; fixed as proposed, now "child row".
- fix-affected-set, LOW: "loom row" twins at `timingspans.go` (`spanOpening` doc and its finding text) and `doc.go` (span opening, Run-Shed fallback); fixed as proposed, now "child row".
- fix-code-correctness: no findings.
- fix-proposal: no findings.
