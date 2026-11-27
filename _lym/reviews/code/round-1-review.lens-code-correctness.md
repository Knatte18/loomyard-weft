```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, tracing each changed branch with a concrete input.

**Cases run (function or branch → case → result)**

| Function / branch | Case | Result |
|---|---|---|
| `Config.PushTiming` | zero value; ceiling below interval | correct |
| `networkWait`, `decideDrive` network row | repair count 0, 4 and large; zero ceiling | correct |
| `noteWait`, `endWait`, `ReadDriveWait` | context cancel; missing directory; free and held lock | correct |
| `loomCLI.failedHalt`, `readRunWait`, `watchRun` | network class under a held driver lock; dead driver | correct |
| `pushledger` `Update`, `HeldRecordsCommit`, `SyncStatus` | empty hub; held-only entry | correct |
| `pushUntilLevel`, `awaitInterval`, `awaitMark`, `waitForPushNow` | push-now mark during an attempt; wake under the 1 s floor | correct |
| `starter.start` with `ifAhead` | level; refused; paused | correct |
| `prepareParent`, `refreshTips` | ledger lost; rebase moves a landing commit | correct |
| `refreshTips`, `pushParent` | rebase removes a recorded landing commit | wrong (F3) |
| `settleOnOrigin`, `dropLandedElsewhere` | `HandOver` or `ResetToOrigin` fails | wrong (F1) |
| `clearRecordedGap` | multi-reason record | correct |
| `restartHubPushers`, `parentTargets` | beam parent with a standing landing and no live pusher | wrong (F2) |
| `landLocally`, `RecordLanding` | first pass | correct |
| `landLocally`, `RecordLanding` | resume after the record | wrong (F4) |
| `syncParent`, `parentPushBlocks`, `needsGitHubOnDisk` | owed; refused; nil seam | correct |
| `keepMergeInBase`, `removeLeftovers`, `RemoveIgnoredLeftovers` | kill after the merge; nested ignore file; check-ignore exit 1 | correct |
| `recordsPushCeiling`, `pushSideDetached`, `PushPairAnchored` ceilings | hold already on origin; no upstream | correct |
| `unmadeOriginCommit`, `PushPrimeCodeTip` | reflog-only commit; moved origin | correct |
| `Fabric.Pull`, `primeAheadByOwedTips` | origin also moved on | questionable (F5) |
| `pushOwedForCatchUp` | prime push owed; pair lock busy; held run | correct |
| `teardownPushSeams`, teardown producer, `LandingStands`, `RemovePair`, `HandOver` | landing standing; dropped; pair gone | correct |
| `primeOfHubHolding` | board directory (main worktree is the records sibling, which `ResolveWithin` maps to the prime) | correct |
| `pushSpawnCommand`, `PusherSync`, `pushSummary`, `syncStatusLine`, `composeSyncStatusCommand` | prime and pair; empty ledger; no hub | correct |

**Findings**

**F1. BLOCKING, design: a failed reset or hand-over is never retried.**
- Location: `internal/runpusher/parent.go`, `dropLandedElsewhere` (about lines 339–368) and the owned-elsewhere branch of `settleOnOrigin` (about lines 279–289).
- Both remove the landing from the ledger before running `ResetToOrigin` or `HandOver`.
- Case: the prime has an uncommitted tracked edit on a landed path, so `ResetParentToOrigin` refuses. The pass returns the error, with the landing already gone and `Dropped[run]` set.
- On the next pass there is no landing to iterate, so neither the reset nor the owned-elsewhere `HandOver` runs again.
- `pushParent` returns early at `len(landings)==0 && len(blockers)==0`, before the `Dropped` blockers are added. No refusal record or notice is written.
- `targetAhead` skips dropped runs, so the target reads level. The duplicate landing commit stays on the local parent, and later landings hit "ahead of its upstream".
- A `HandOver` that fails in `settleOnOrigin` (for example an offline push) is lost the same way.
- Evidence: `parent.go` as shown in diff piece 034.

**F2. BLOCKING, design: the dobby tick never starts a beam parent's pusher.**
- Location: `internal/beamcli/dobbydeps.go`, `restartHubPushers` (the `for _, target := range pairs` loop) with `parentTargets`.
- Pair targets that hold landings are only passed to `settleIfRefused`; `start` is never called for them. The only other tick-time start is `DobbyRuns`, which covers batten runs' pairs, and a beam pair is not one.
- Case 1: a beam parent's push is refused, and the operator runs the named `lym fabric push`. It lands and clears the code record, so the target no longer reads refused. Nothing then settles or starts it: no `done` write, the member's records stay held, its teardown refuses, and the all-clear never posts.
- Case 2: the beam pusher stops on a reboot, pause or guard stop while the push is owed, and nothing restarts it.
- D3 requires that the next dobby tick drops the entry or starts the pusher.
- Evidence: `dobbydeps.go:245-266`; `battencli/dobbyreads.go:324`.

**F3. MEDIUM, design: a landing commit that a rebase removed is still pushed.**
- Location: `internal/runpusher/parent.go`, `refreshTips` (about lines 238–255) and `pushParent` (about lines 496–521).
- `refreshTips` updates a landing's `Tip` only when `LandingCommits` still lists the run. When a rebase in the parent worktree removed the commit and origin does not hold it, the old SHA stays in the entry.
- `pushParent` never checks that the local head carries `landing.Tip`.
- Case: for the prime, `PushPrimeCodeTip` pushes the orphaned SHA. It is a fast-forward of origin, so the commit the operator removed lands on origin and the run is marked done.

**F4. MEDIUM, design: a Finalize resume releases the originally held records commit.**
- Location: `internal/loomcli/landingdeps.go`, `recordHeldLanding`, with `RecordLanding` in `parent.go`.
- Case: the first pass records hold C1, then Finalize is killed. The resume commits a newer status C2, or reads a newer HEAD.
- `RecordLanding` deletes the run's old landing and stores C2. The next push may then send C1 up to C2's parent, while origin's parent still lacks the run's work.
- D5 says neither the held commit nor any after it is sent while the entry stands. The replacement should keep the earlier unpushed `RecordsCommit`.

**F5. LOW, design: a diverged prime pulls "successfully" without moving.**
- Location: `internal/fabricengine/pull.go`, `primeAheadByOwedTips`, as used in `pullWarpArm`.
- It checks only that every local-ahead commit is an owed tip, not that origin has nothing new.
- Case: one owed tip locally, plus a new commit on origin from a PR merge. The pull returns nil with the prime not updated, so batten's Land deploys code that lacks origin's commit and reports no stop.

**Tool calls**
1. `state.ReadJSON` and the lock package: needed to check that wait-record reads cannot create directories under a removed pair (they do not).
2. `lymcwd.ResolveWithin` and `PrimeName`: needed to check how `primeOfHubHolding` resolves a records worktree.
3. `startPusher` uses and the `DobbyRuns` loop: needed to confirm no other tick path starts a beam parent's pusher (F2).
