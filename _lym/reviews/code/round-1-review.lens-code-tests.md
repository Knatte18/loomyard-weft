```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

## Behaviour → pinning test

| Behaviour | Test |
|---|---|
| Push timing defaults and floors | `TestConfig_PushTiming` |
| Driver network wait, uncapped re-step, wait record lifecycle | `TestDecideDrive`, `TestGoDriver_Settle` |
| loom and batten driver deps fill `NetworkWaitCeiling` and `WaitRecordPath` | **none** (F9) |
| `ReadDriveWait` | `TestReadDriveWait` |
| No halt note under a live driver; status `waiting` | `TestLoomPostRun_HaltNotes/NetworkStop`, `TestLoomStatusExtras` |
| Run watch: waiting run, dead driver | `TestRunWatch` |
| `DobbyRuns` reads a batten wait in the prime | `TestDobbyRuns_…` |
| `DobbyRuns` reads a pair's wait and the gap before the record is written | **none** (F8) |
| Ledger writes, clear time, held records | `pushledger` tests |
| Pusher interval, doubling, mark, level entry, cancel | `TestRun`, `TestNewDeps_WaitWakesOnAPushNowMark` |
| Single transport attempt | `SingleAttemptOverAnUnreachableOriginTriesOnce` |
| Configured interval reaches the pusher | **none** (F9) |
| Board and landing targets, spawn line, ledger entry, prune | `TestStart`, `TestPruneLedger`, `TestPusherCmd_…`, `TestPusherDeps_BoardPush…` |
| Board target reads a real refused record | **none** (F6) |
| Spawns start the pusher | `TestSpawnStatusPush`, `TestPushSpawnCommand`, `TestSpawnSync_…` |
| Owed code commits and held records read level | `TestRunCommitsAhead` |
| Records hold, code ceiling, removal | `TestPushPairAnchored_…`, `TestPushRun_StopsShortOfAHeldRecordsCommit`, `TestHubLandDeps_PushBeam…` |
| A rejected capped push is never replayed or reset | **none** (F4) |
| Prime code push, parent-ahead reads, held-back way forward | `TestPushPrimeCodeTip`, `TestParentAheadReads`, `TestFailedPushWayForward` |
| `ResetParentToOrigin` refuses a dirty path or a branch with no upstream | **none** (F7) |
| Parent pass: settle, prime owned elsewhere, drop and reset, rebase refusal, unrecorded and lost landing, beam run elsewhere, beam happy path | `TestParentPush` (7 subtests) |
| Owners pulled from origin before any owner read | **none** (F1) |
| Settle or drop of a run owned elsewhere (no `done` write, hold, hand-over) | **none** (F2) |
| Partial and whole hold-back | **none** (F3) |
| Dobby settles a refused target | `TestRestartHubPushers` (fake settle only) |
| Local landing, resume, ahead-by-owed, trailer commit, `landLocally` lock | `TestLandingDeps_LocalLanding`, `TestLandLocally` |
| Pushes off records no landing | **none** (F5) |
| Owner check and tips read are `git-transport` | `TestFinalize_OwnerChecks` |
| Pull-request landing while a push is owed or refused | `TestLandingDeps_ParentPushState` |
| Merge-in leftovers | `TestFinalize_RemovesIgnoredLeftovers…` |
| Past-the-merge read | `TestReadRun` |
| Prime pull and Land transient | `TestLandDeps_PullWithAnOwedLandingPush`, `TestLand_Dispositions` |
| Catch-up hand-off | `TestPushOwedForCatchUp` |
| Reset guard | `TestRejectedCodePush_Refuses…` |
| Teardown, cancel, hand-over, remove, leftover refusals | `TestTeardown_CallSequence`, `TestEndForCancel`, `TestCommand_RemoveRefuses…`, `TestCancelIntegration_…`, `TestTearDownLeftovers_…` |
| Push summary logic and wired waits | `TestPushSummary`, `TestHubDobbyDeps_PushSummaryWaits…` |
| Push summary's wired refusal read and way forward | **none** (F6) |
| `SyncStatus` | `TestSyncStatus` |
| `lym fabric sync-status` verb | **none** (F10) |
| Bar pin | `TestPinBindingsLocked_…` |

## Findings

**F1 — BLOCKING, scope.** `internal/loomcli/parentpush_integration_test.go:234-405`; code `internal/runpusher/parent.go` (`prepareParent` Refresh).
No row puts an owner entry on origin's board only. Removing the Bolt pull or beam reconcile before a pass or `SettleRefused` would fail no test. Card 11 and added entry "round-3 findings", item 3, require this row. Every `setOwners` call writes the local board.

**F2 — BLOCKING, scope.** Same file; `parent.go` `settleOnOrigin` (`owner != ""`) and `dropLandedElsewhere` (`HandOver`).
No test settles or drops a landing whose run another machine owns. Unpinned:
- no `done` write;
- records kept as `HeldRecords`;
- hand-over;
- no refused record;
- origin never receives the held commit.

The arbiter entry's F3, D5's "for a run owned elsewhere the pusher hands it over" criterion and card 11 all name these rows. The card's row for a beam owner's landing under another commit, dropped by `SettleRefused`, is missing too.

**F3 — MEDIUM, scope.** `parent.go` `pushParent` (`blockers`, `wholeHeld`).
No test shows that an older tip not carrying the held run is still pushed and settled. None shows that a beam parent carrying a run owned elsewhere is held back whole. `ABeamParentTakesTheLandingOnItsOwnEntry` is the happy path only. Card 11 also names these missing rows:
- the beam-parent "only landing dropped, then rebase, then `SettleRefused` clears" row;
- "pair removed after a drop with a newer landing: later passes neither re-record nor push".

**F4 — MEDIUM, consistency.** `internal/fabricengine/pushclassified.go` (`retryWeftPush` ceiling branch; `j.opts.ceiling != ""` in `repushOverOriginContained`).
No row rejects a capped push against a moved origin. If a replay rewrote the held records commit, or a reset fired under a code ceiling, no test would fail. All the hold and ceiling rows push to an origin that has not moved.

**F5 — MEDIUM, scope.** `internal/landingshed/finalize_test.go`, `TestFinalize_MergeMarkPushOrderAndVerdict`, "pushes off" row.
`LandLocally` is nil there, and `pushParent` takes the same branch for `LandLocally == nil`. Dropping the `PushSkipped ||` condition keeps the row green. D5's criterion "with git or pushes off a local landing writes no landing entry" is unpinned. The row needs a recording `LandLocally`, or a real-wiring row.

**F6 — MEDIUM, scope.** `internal/beamcli/dobbydeps.go` (`refusedPushWayForward`, `recordNamesTarget`); `internal/runpusher/targetreads.go` (`boardRefused`).
No test references any of them. `TestPushSummary` fakes `Refused` with a map, and `TestStart` fakes the refusal read. Unpinned:
- D5's "a refused landing push reads as refused in D3's summary through the prime's code-branch record";
- D3's "after the way forward clears it … the next tick drops the entry and the all-clear posts", over the real reads;
- a refused board record stopping the board pusher.

**F7 — LOW, consistency.** `internal/fabricengine/destroy.go` `ResetParentToOrigin`.
This is a new destructive executor. Only its success path is tested (`TestParentAheadReads`). The dirty touched path and no-upstream refusals, with nothing moved, have no row.

**F8 — MEDIUM, consistency.** `internal/battencli/dobbyreads.go` `readRunWait`.
The new `DobbyRuns` rows cover only batten's own record in the prime. Neither a pair's wait record nor the "failed network status under a held driver lock, no record yet" branch has a row. Breaking either would make the run watch mail a waiting run as stopped. D6 and card 3 require that it does not.

**F9 — LOW, consistency.** `internal/loomcli/drivewiring.go`, `internal/battencli/drive.go`, `internal/loomcli/pusher.go` `pushTimingOf`.
No test asserts that the drivers' deps carry `fabric.push_ceiling_s` and the wait record path. `run_test.go` only seeds a config so the load succeeds. `TestPusherDeps` runs only the no-config default. Dropping any of this wiring would fail nothing, and a zero ceiling silently caps every wait at 15 s.

**F10 — MEDIUM, scope.** `internal/fabriccli/syncstatus.go`.
D4's criterion pins `lym fabric sync-status`'s output. Only `pushledger.SyncStatus` is tested. The verb's printed line, `sync: ?` with exit zero on an unreadable hub, the empty line for a non-hub and the base interval read from the prime's config have no row. The verb appears only in the arity and role-guard tables.

Tool calls:
1. grep of the `TestParentPush` subtests and of test references to the wiring symbols: the context showed diffs only, so it could not show that no test elsewhere references them.
