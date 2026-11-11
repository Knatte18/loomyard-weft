# Round 1 fixer-report

## Changes

- F1, fixed as proposed, then tightened after the self-check:
  `internal/heddleengine/replayer_test.go` gains `UnversionedSidesReadClaudeFileAsAgentsFile`
  and `UnversionedMergeBaseReadsClaudeFileAsAgentsFile`.
  Both feed unversioned heddle.json bytes through `Replayer.Apply`.
  The first pins the parent and commit sides, with an origin whose targets differ so an unmigrated side shows as lost.
  The second pins the merge base, where the commit's change applies only when the base reads as agents-file.
  Commits `bad97e8a` and the one after `1f2c65f5`.
- F2, fixed as proposed:
  `internal/heddleengine/store_test.go` gains the rows `AgentsFileBesideClaudeFile` and `NewerVersionedClaudeFile`,
  and a Save round-trip that keeps version 2.
  The subtest is renamed `StoreVersionDecidesHowClaudeFileIsRead` since it no longer covers only unversioned stores.
  Commits `1f2c65f5` and the one after it.

The gate, `lym gate test ./internal/heddleengine`, passes.

## Self-check

- fix-code-correctness, LOW: the replayer test pins only the commit side. Fixed with my own change, a second subtest for the merge base and a differing origin for the parent.
- fix-proposal, MEDIUM: same finding as above, merged with it. Fixed the same way.
- fix-consistency, LOW: the subtest name no longer matches its rows. Fixed by renaming it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
