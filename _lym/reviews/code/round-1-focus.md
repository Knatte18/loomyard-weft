---
round: 1
focus:
  - "internal/boardengine: does any path other than Store.UpsertTask, UpsertTasksBatch and MergeTasks still write to or delete from a map its caller owns? Check the Board wrappers (UpsertTaskReporting, UpsertTaskWithRecord, FileFrontTask, MergeTasks, UpsertTasksBatch) and the payload check helpers' other callers."
  - "Callers of the three store methods and their Board wrappers (boardcli, beamcli, retrocli, intake, front, raddlefile, siblings, recordupsert): did any rely on the former in-place key deletion or on keys MergeTasks added to the map?"
  - "internal/boardengine/store.go: is a shallow clone enough, that is, does anything on these paths mutate a nested slice or map value of a payload? Do nil payloads and nil batch elements behave as they did before?"
  - "internal/boardengine/store.go, MergeTasks: with the clone moved above the checks, trace what the final UpsertTask call receives and whether the merge result is unchanged, including when the upsert carries keys the checks drop."
  - "internal/boardengine/decision_test.go, TestDiscussedPayload: does the unchanged-payload assertion fail on the base code for each path, and is its expected value taken independently of the writes by the parallel subtests? Does any other unsynchronized shared state remain in the test or the code it drives, given the entry asks for the race to be found with the race detector?"
  - "_lym/landing/summary.md and the commit message: check each claim against the diff and the base code, including the stated failure on the old code and the Merge danger door and blast radius."
  - "The Root-Cause trailer and the changed doc comments on the three store methods: do they match what the code does?"
---

Seed for round 1.
The range holds one commit, touching `internal/boardengine/store.go` and `internal/boardengine/decision_test.go`.
The decision record carries only the board entry under `## Goal`, so that entry is the measuring stick.
