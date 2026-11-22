---
round: 4
focus:
  - "Decision D4 step 2 in `_lym/discussion/decision-record.md`: where does `lym beam ready` leave `_lym/landing/beam-ready.json` (committed by `MarkReady` in `internal/beamcli/deps.go` through `fabricengine.CommitAnchoredPaths`, which commits in `RecordsWorktree` per `internal/fabricengine/commitweftpaths.go`), and does the step's read of that record, local and for a beam on another machine, look in that repository and branch? Settle whether an absent record on origin can wrongly let a flagged beam pass the check, and whether the step states what bounds that."
  - "The D4 acceptance bullet and the support-log line that name the branch holding the ready record: do they match the branch D4 step 2 reads?"
---

Only the D4 step 2 read and the lines that restate it need a look; the rest of the record settled in earlier rounds.
