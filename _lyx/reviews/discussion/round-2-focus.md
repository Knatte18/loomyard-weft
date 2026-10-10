---
round: 2
focus:
  - "Check every claim in the decision record that a count or budget bounds a routing edge against how `episodeStuckCount` in `internal/shedengine/run.go` ends a producer's episode at a Done by any producer sharing its segment: does the stated bound on the PR-Gate to Darn edge match what the engine enforces, and does each Decision introducing an edge state what it can let through?"
  - "Check each Decision that clears or releases a board claim or run lock: does it state which run states (landed but unmarked done, still-live batten run, halted run) it lets through, and what bounds that, given `RunLockedError` in `internal/boardengine/runlock.go` and `Finalize.markTaskDone` in `internal/landingshed/finalize.go`?"
  - "Check any text planned for packages outside the `PATTERN-fabric-vocabulary` owner set (`fabricengine`, `fabriccli`, `weftname`, `gitkit`, `hubforge`, `boardengine`, `configsync`), such as `landingshed` Stuck messages and `loomcli` halt notes: does it name warp or weft sides, and does it agree with the record's own vocabulary constraint?"
  - "Check each config value the record makes a told reader: is an empty or missing value refused at every call site that reads it (gate, Publish, Finalize), or only at one entry point, and does an acceptance criterion pin that?"
---

Entries restate their sites so the reviewer can settle each question without earlier rounds.
