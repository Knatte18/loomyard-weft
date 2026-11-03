---
round: 1
focus:
  - "internal/battencli/wire.go WriteSeed seam: does writing the child-recipe record after the child seed leave a consistent state when the write fails or the seed step reruns, and is `recipe` the value the child run was actually seeded with, for every recipe path through Seed-Child?"
  - "internal/battencli/cleanup.go clearMotifRun against cleanup: does it take the same locks in the same order, judge done with the same `judgeStatus` meaning, and call `removeRunRecords` the way cleanup does, so no run started meanwhile and no non-done run is removed?"
  - "internal/battencli/cleanup.go clearMotifRun error handling: which failures return an error, which only warn, and does `cleared` report true only when the records are truly gone? Check the scratch-empty and push-spawn paths."
  - "internal/beamcli/create.go createSteps: is the `found && state == Done && !PairPresent` guard the right trigger for a torn-down motif pair, and does the flow after a clear (stale `found`/`state`, the `RunLive` branch, the StartRun call) still behave correctly? Check that a failing or false-returning ClearMotifRun leaves the old refusal intact."
  - "internal/battencli/commitstatus.go battenRunCommitPaths with the child-recipe record: is the record committed in the status commit that lands it and removed by `removeRunRecords`, so no stray tracked file remains in the records repository?"
  - "Scope against the decision record's Goal: the Expected section allows clearing at create or removal at landing. Does the diff go beyond clearing a done motif run (any other done run still refused), and does it cover a motif run whose child recipe record is missing, as the change description claims?"
  - "Docs the repository names for this kind of change (package doc.go files, the batten and beam CLI docs, any spec under the hub's specs directory): does each land in the diff, and does the change description's claim about each file match the diff?"
  - "Test economy: do `TestClearMotifRun_RemovesOnlyADoneMotifRun`, the new `TestCreate` rows and `TestChildRecipe` duplicate existing coverage, and does any behavior the diff adds (the WriteSeed write, the warn paths in clearMotifRun) have no test?"
---

Seeded from the decision record's Goal, the commit message, the change description and a read of the non-test diff.
