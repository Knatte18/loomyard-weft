---
verdict: APPROVED
---

Round 2 reviewed `git diff df15bb6a9b153f92eca9f40276be787e80f60f3e..HEAD` against the decision record's D6 and D10 and its acceptance criteria.
The commit messages carry no `Pattern-Departure:` trailer, and the directive declares no PATTERN change.
No finding was raised.

## Focus answers

- `TestFileRaddle` tasks-shape (`internal/boardengine/raddlefile_test.go`):
  the row `TasksShapeMovesOpenDependentsOntoTheSinkPartAndLeavesDoneOnes` re-points `waiter` from `[other, plan]` to `[other, plan-b]`.
  It leaves the done dependent `finished` on `[plan]`.
  The family subtest pins the multi-sink case, `[other, plan-b, plan-c]`.
  The one-task re-point is therefore covered.
- `internal/loomshed/raddlefile.go` `Call`:
  - A filing found as its own (store done, or `AlreadyFiled`) is repaired before any other step.
  - The repair blanks the local `entry.StatusProducer`, so `stageAndFile` never claims a status back onto the project.
  - `finish` is reachable only after `parseErr == nil`, because `Call` returns `fail("parse the proposal")` first.
    `EndEntry` therefore never runs on an unparsable proposal, and `family` is skipped in `finish` on every path.
  - The tests `AFamilyFilingFoundAsItsOwnIsRepairedAndNotFiledTwice` and `AFailingMarkOrStatusClearStopsTheRepairBeforeAnythingElseRuns` pin the repair and the claim seam.
- `internal/boardengine/raddlefile.go` `endsAsFamilyProject`:
  - It refuses a kept `profile`, `run_settings` or `notify_on_land` explicitly, since `scopeChanged` does not read those three.
  - The rows `AKeptProfileIsRefused`, `KeptRunSettingsAreRefused` and `AKeptNotifyOnLandIsRefused` in `TestCheckRunLocks_FamilyProject` reach each branch.
  - `scopeChanged` against `arrasProjectOf` covers the remaining cleared fields.

## Rejected

- The re-point skips done dependents, where D10 says "every other entry".
  The change description names this, and a done entry waits on nothing, so the skip is not a defect.
- The mark is written with `primeOfHub` and read with `primeOfHub` plus `shedrun.ResolveRunID`, while `raddleFileDeps` takes the already resolved slug.
  Both resolve to the same run directory, so there is no mismatch.
