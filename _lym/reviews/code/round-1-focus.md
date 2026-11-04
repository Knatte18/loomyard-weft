---
round: 1
focus:
  - "internal/sheddriver/drive.go: the yield early-return now precedes SaveEnvelope. Settle whether any path that previously read the saved envelope of a yielded pass (resume, report, repair, budget, tests of the stop envelope) still works, and whether `resumed = false` and the other post-save state are still reached or no longer needed."
  - "internal/battencli/drive.go and commitstatus.go: `commitRunRecords` runs in the Park seam and the CommitRecords seam. Settle whether Park still parks the run on every commit failure, whether a stale or empty pathspec can make the commit fail, and whether committing at a done stop runs after the last envelope is saved."
  - "internal/shedrun/paths.go `DriverRecordsRel` and `dirHoldsFile`: settle whether the walk swallowing every error can hide an unreadable directory that then stays untracked, and whether the pathspec result matches the three inlined checks it replaced in loomcli/wiring.go (same directories, same order, same emptiness rule)."
  - "Scope against the Goal: the issue asks that everything under `_lym/` be tracked or moved to `.lym/`, and asks whether a report is written per poll. Settle whether the diff covers loom and batten drivers alike, and whether any driver directory under `_lym/shed/<slug>/` (for example agent attempts the batten commit message mentions) is still left untracked."
  - "Test coverage: the new `commitRunRecords` branches (merge active, nothing committed, spawn failure, commit error), the changed yield behavior in sheddriver/drive_test.go, and `DriverRecordsRel`. Settle which behavior has no test and which new test duplicates an existing one."
  - "Comments: new and changed doc comments in the five touched non-test files follow the repository's comment conventions, and the `statusCommitPathspec` doc comment matches the code after the edit."
---

Seed pass: focus is drawn from the committed diff and the Goal in the decision record, which holds only the imported issue.
