# Card 5 — finalize-lock-free-merge

**Create:**
- `plan:internal/landingshed#Finalize.mergeUnlocked` -> `func (fz *Finalize) mergeUnlocked(ctx context.Context, front bool) *ending`
- `plan:internal/landingshed#TestFinalize_MergesWithNoLockAndGatesUnderTheLock` -> `func TestFinalize_MergesWithNoLockAndGatesUnderTheLock(t *testing.T)`

**Edit:**
- `internal/landingshed#Finalize`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed#Finalize.freeLock`
- `internal/landingshed#tipRecordingResolver`
- `internal/landingshed#tipRecordingResolver.Resolve`
- `internal/landingshed#TestFinalize_LockFreePassStops`
- `internal/landingshed#TestFinalize_LockFreePassMergesTheUpstreamTipOfAnAheadParent`

**Delete:**
- `internal/landingshed#Finalize.gateUnlocked`
- `internal/landingshed#TestFinalize_GatesWithNoLockAndLocksToLand`
- `internal/landingshed#TestFinalize_QueuesAtTheFrontAfterALostRace`
- `internal/landingshed#TestFinalize_FixerInTheLockFreePass`

**Uses:**
- `internal/landingshed#recordingParentMerger`
- `internal/landingshed#newTestDeps`

**Intent:** Remove Finalize's lock-free gate, its lost-race loop and the gated-tip recheck (D1); the lock-free merge stays.

- `Finalize.gateUnlocked` is deleted; its one caller is `Finalize.Call`.
  `mergeUnlocked(ctx, front)` takes its place and does what it did up to the merge: the held read (a parent holding the work returns nil at once), the owner checks, `readConfigBeforeMerge`, the `needsGitHubOnDisk` read and `parentMergeTip`, then `mergeParent` through the resolver.
  It runs no gate and keeps no tip.
- `Finalize.Call` runs `mergeUnlocked` and then `landUnderLock` once per pass; the `lostRaces` count, the `Config.FinalizeLostRaces` comparisons and their log line go.
  The loop stays only for a pass that freed the lock for a fixer or fix review (`lockFreed`), which runs the passes again at the front, as today.
  The doc comment drops every sentence about the lock-free gate, lost races and `landing.finalize_lost_races`, and says the merge runs with no lock and the gate only under it.
- `Finalize` loses its `gatedTip` and `parentMoved` fields.
- `landUnderLock` loses step 1b4b, the head comparison with `gatedTip`, and its doc sentences about it; under the lock it merges the parent again, as today, which finds a parent that did not move up to date, and gates there.
- `freeLock`'s doc drops "The lock-free pass runs first while lost races remain."
- `finalize.go`'s file comment, lines 1-4, is rewritten: with no parent lock held Finalize merges the parent's tip into the task worktree; under the parent lock it merges again, gates the merged tree and lands. Card 12 rewrites `doc.go`'s Finalize paragraph.
- `Config.FinalizeLostRaces` is left in place, unread, for card 6 to remove with its key.

The file `finalize_unlocked_test.go` keeps its name; its header says it drives the lock-free merge and the gate under the lock.
The tests of the lost race go, since the behaviour goes: `TestFinalize_GatesWithNoLockAndLocksToLand`, `TestFinalize_QueuesAtTheFrontAfterALostRace` and `TestFinalize_FixerInTheLockFreePass`, whose fixer now runs only under the lock, where `TestFinalize_GateAndFixer` covers it.
`TestFinalize_MergesWithNoLockAndGatesUnderTheLock`, tier0 in `package landingshed`, takes over the kept rows of the deleted table over `tipRecordingResolver`, `recordingParentMerger` and a real `parentlock.Store`, adding about 1 second to one tier0 run.
`tipRecordingResolver` gains `onResolve func(call int)`, called at each `Resolve` with its 1-based count; nil calls nothing; a row moves the local parent from it.
Each row asserts whether the lock was held at each verify, the tip of each merge of the parent and the parent pair's calls:
an unmoved parent merges `p1` with no lock and `""` under it, with the one verify under the lock; a parent that `onResolve` moves to `p2` during the lock-free merge is merged `""` under the lock, the merge under the lock bringing in `p2`, and verifies once, under the lock; a parent already holding the work runs no lock-free merge; a parent behind its upstream merges `u1` with no lock and is fast-forwarded under it; a parent ahead by owed tips merges `p1` with no lock.
`TestFinalize_LockFreePassStops` drops its lost-race setting and keeps its rows on the owner checks and stops before the lock-free merge.
`TestFinalize_LockFreePassMergesTheUpstreamTipOfAnAheadParent` drops its lost-race setting and keeps its assertion that a landing known to need GitHub merges the upstream tip.

**ImpactSummary:** One production caller of the deleted method (`Finalize.Call`); the change stays inside `internal/landingshed`'s Finalize, its file comment and its unlocked-pass tests and fake.
