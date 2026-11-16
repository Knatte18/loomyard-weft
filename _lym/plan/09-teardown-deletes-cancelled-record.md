# Card 9 — teardown-deletes-cancelled-record

**Create:**
- `plan:internal/pairteardown#TestRemovePair_DeletesACancelledRunsBattenRecord` -> `func TestRemovePair_DeletesACancelledRunsBattenRecord(t *testing.T)`

**Edit:**
- `internal/pairteardown#Teardown.RemovePair`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `plan:internal/runfsm#ReadIntents`
- `internal/shedrun#RunDir`
- `internal/shedrun#RunDirRel`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#SpawnStatusPush`
- `internal/pairteardown#TestRemovePair_ClearStray`

**Intent:**
Every path that removes a cancelled run's pair deletes the slug's batten record in the same step (D3).
`Teardown.RemovePair` is the step `lym fabric remove <slug>` and the dobby's teardown reach, and the cancel's own `--remove` reaches it too.
After it removes the pair, when the prime's batten record of the slug holds a cancel intent (`runfsm.ReadIntents`), it removes `shedrun.RunDir` at the prime and commits the removal with `shedrun.RunDirRel` on the prime's records side, then spawns the status push as the teardown's other records pushes do.
A record with no cancel intent is left as today, so removing the pair of a run that was never cancelled changes nothing.
The doc gains a line on the deletion.
`TestRemovePair_DeletesACancelledRunsBattenRecord` (tier1) removes the pair of a run with a finished keep cancel and finds its record gone and the removal committed, and leaves the record of a run without a cancel intent.

**ImpactSummary:** pair removal also deletes the batten record of a cancelled run; no other run's record is touched.

**Commit:** 9: Removing a cancelled run's pair deletes its batten record
