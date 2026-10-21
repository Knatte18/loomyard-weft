# run-reentry: stuck before batch 12

Four of the twelve batches are done; the run stopped at batch 12 on a plan conflict.

## Batches done

- **01-state-write-file** (cards 1–4): `record-batch` first failed it, because the Create target `plan:internal/state#TestWriteFile` did not resolve.
  One `recover-batch` run finished it as `done`, at head `1b97c2fd`.
  Scope-guard deviations: `internal/shedengine/goto_test.go`, `internal/shedengine/gotobudget_test.go`, `internal/shedengine/run.go`, `internal/shedengine/run_routing_test.go`.
  No departures.
- **05-shedverbs-pause-label** (cards 5, 6): done at head `9534bf85`.
  Scope-guard deviations: `internal/sheddriver/driverules.go`, `internal/shedverbs/pause.go`.
  No departures.
- **07-review-amend-key** (cards 7, 8): done at head `0fd9f982`.
  Scope-guard deviations: `internal/fabricengine/taskpatch.go`, `internal/fabricengine/taskpatch_integration_test.go`.
  No departures.
- **09-identity-text** (cards 9, 10, 11): done at head `92e1a7d1`.
  Scope-guard deviations: `internal/discussionparser/identity.go`, `internal/shedadapters/bouncer.go`, `internal/shedadapters/bouncer_reentry_test.go`, `internal/shedadapters/settlerecord.go`, `internal/shedadapters/settlerecord_test.go`.
  No departures.

## Where it stopped

`begin-batch 12` refused with `plan_conflict: tree_moved`.
The refusal named the Create target `plan:internal/shedadapters#Bouncer.startGeneration`, which already resolves in the tree.
Batch 09 added that method while implementing card 11.
Per the run rules the verb was not retried and no later batch was begun.
The way forward is a plan edit: edit card 12 so it matches the tree, run `lyx webster rebaseline --card 12`, then run `lyx webster run` to resume.

Batches 14, 15, 16, 17, 18, 19 and 20 were not begun.
