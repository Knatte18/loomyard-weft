---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-landing-pre-merge-clear.md
    summary: a strand-table read failure in conflictSessionStopper reaches preMerge.clear as a stop error with an empty guid, so the Stuck reason names strand () and lyx reed remove with no guid, and no premerge_test row reaches that exit
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/06-resume-start-own-leftover.md
    summary: refuseOverUnfinishedMerge reads the parent branch for every parked state before checking producer and verb, so a read failure at any other row replaces the merge_in_progress refusal D3 says those rows keep with a plain error
---

### [BLOCKING] An unreadable strand table becomes a Stuck reason with an empty guid

**Location:** `_lyx/plan/05-landing-pre-merge-clear.md`, the `Deps.StopConflictSession` contract, `preMerge.clear`'s own-leftover branch and the `conflictSessionStopper` paragraph.

**Issue:** The card defines `StopConflictSession` as returning an empty guid with a nil error when every stop succeeded or none was live, and the failing stop's guid beside its error otherwise.
`conflictSessionStopper` reads reed's strand table at call time, and its test row says "a status error is returned"; with no strand in hand, that return has an empty guid beside a non-nil error.
`preMerge.clear` maps any stop error to the Stuck reason `a conflict session of this run (strand <guid>) is still live and could not be stopped: <err>; way forward: run "lyx reed remove <guid>", then "lyx loom resume"`.
So a failed `reed.Status` read stops the row Stuck claiming a live session it never found, with `strand ()` and the way forward `lyx reed remove ` naming no guid.
That way forward cannot be followed, which breaks `PATTERN-refusal-way-forward`, and D2 licenses the guid-naming Stuck only for "a stop that returns an error", not for a strand-table read fault.
No `premerge_test.go` row covers a stop seam error with an empty guid, so nothing would catch it.

**Fix:** In `preMerge.clear`, treat a `StopConflictSession` error with an empty guid as an infrastructure error, returned as an error that both producers' `gateStop` maps to a cancellation or a fault, with the merge left untouched and no abort.
Keep the guid-naming Stuck reason for an error with a non-empty guid.
Add a row to the planned `premerge_test.go` table: the stop seam returns an empty guid and an error, and the row asserts an error return, no `AbortMerge` call and no merge-in.
The row extends the card's one planned table, so it adds no target and no top-level test beyond the budget rise the card already lists.

### [LOW] The parent-branch read can turn another row's merge_in_progress refusal into a plain error

**Location:** `_lyx/plan/06-resume-start-own-leftover.md`, the `refuseOverUnfinishedMerge` paragraph and the `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` row.

**Issue:** The card has the guard read the parent branch through `recordedParentBranch` "for a parked state", which is before `ownMergeInLeftover` checks the producer and the verb, and has a read error refuse like a probe error.
At a row other than Publish or Finalize, or for a parked `merge`, a read failure therefore reports a plain error with no `merge_in_progress` kind and no `conflicts` list, though D3 says every other row and every other verb keep that refusal.
The planned probe-error row names no producer, so it does not pin which order the guard reads in.

**Fix:** Read the parent branch only when the state is parked, `Verb` is `merge-in` and the producer is `loomshed.NamePublish` or `loomshed.NameFinalize`, and let every other parked state take today's refusal without the read.
Have the `ProbeErrorRefuses` row set the run's current producer to Publish, so the read error is reached on the path that needs it.

## Focus directives

1. Card 5's `AbortMerge` error exit is reached: the planned `premerge_test.go` table has the row "a stop that succeeds followed by a failing `AbortMerge` stops Stuck naming the parent branch, `lyx fabric merge --continue` and `lyx fabric merge --abort`, with no path list and no merge-in (the fake resolver is not called)".
   The `Publish or Finalize: merge in progress` spec row covers that reason.
   Not a defect.
2. Card 6's narrowed guard's new error exit is reached: `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` gains the row "a parked `merge-in` state with a `recordedParentBranch` seam that returns an error refuses with a plain error carrying that error, puts nothing to work and releases the bootstrap lock".
   `refuseOverUnfinishedMerge` is the one guard both `start` and `resume` call (`internal/loomcli/start.go:141`, `internal/loomcli/resume.go:102` and `:118`), so one row through `runDriverSpawnAndWait` reaches the exit for both.
   Not a defect beyond F2, which concerns where the read sits, not whether it is tested.
3. Budget and verify: `cmd/lyx/testdata/test-budget.yaml` has `internal/landingshed: 66` and `internal/loomcli: 182`, and each package has exactly that many top-level tests today.
   Card 5 adds one top-level test to each, `premerge_test.go`'s table and `TestConflictSessionStopper`, raises both rows by one and lists the file under `Edit:`.
   Cards 2, 3, 4, 6, 7 and 8 add rows or assertions to existing tests only, and F1's fix adds a row, not a test.
   The `## verify:` section runs `go test ./...` and `go test -tags integration ./...` and vets the `tmux` and `llm` tags, which covers every package a card targets, including `contracts/stencils` and `internal/shuttleengine`.
   Not a defect.
