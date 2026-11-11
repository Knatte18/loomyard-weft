```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `runAdopt` exempts the gate once before any Bolt pull and restores it once | `TestRunAdopt`: `fakes.exempted`/`restored` asserted in every row, plus a pull-before-exempt check in `PullBolt` (`caller_test.go`) | pinned |
| `runWorker` exempts before its first step (reconcile) and restores on return | `TestRunWorker` row `WorkerPassesEveryPauseFromBeforeItsFirstStepUntilItReturns` (`worker_test.go`) | pinned |
| Production wiring of `ExemptPause: pauserecord.SetGateExemptAll` in `hubCallerDeps` and `hubWorkerDeps` | none; tests use fakes | gap (finding 2) |
| `waitRecordsLocks` polls while only the push lock is held, and the weft moves once it is released | `TestReconcileOrigin_Moves`, push-lock-wait row, released at sleep 2, `BranchLevel` | pinned |
| The wait stays bounded and the lock still held past the bound reads `BranchBusy` | same row, `releaseAt` 0, 3 sleeps, `BranchBusy` | pinned |
| A zero `PushLockWait` reads a held lock busy at once | existing busy row of `TestReconcileOrigin_Moves` runs with zero options | pinned |
| A held write lock does not wait, even with a nonzero bound | none; the new row holds only the push lock | gap (finding 3) |
| The adopt passes `reconcilePushLockWait` through `hubDepsAs` → `ReconcileOrigin` | none; my grep finds no test reading the constant or the deps wiring | gap (finding 1) |
| `DropLocal` removes an untracked file origin's tip tracks, after the tag and before the reset, and records `path_removed` | `TestDropLocal` subtest "An untracked weft file origin's tip tracks…" checks HEAD, content, and kinds `tag_created, path_removed, commits_dropped` | pinned |
| An ignored untracked file is left alone | none; `droplocal_integration_test.go` has no ignore row | gap (finding 4) |
| A code branch's untracked files are never removed (`side == failedPushSideRecords` guard) | none | gap (finding 4) |
| An untracked file origin's tip does not track is left alone | none | gap (finding 4) |
| `ownedUntrackedTrackedAt` refuses a file the index tracks or one the commit does not track | none; `TestGate_AbsentTargetIsNoOp` covers only the absent-target no-op | gap (finding 4) |
| The failure path of the removal or reset wraps the error and names the archive tag | none | LOW, covered by finding 4 |
| `rejudges`: a record stopped at `done` restarts at reconcile with steps cleared, and a second reconcile ends done | `TestRunWorker` row `HeldBackRecordWhoseStopNoLongerHoldsIsJudgedAgainAndEndsDone` | pinned |
| A record stopped at another step still resumes there | `worker_test.go:220` (deploy stop), `adopt_test.go:1019` | pinned, existing |
| A held-back record whose stop still holds on re-run reaches the operator with the `lym adopt --discard` way forward | none for the re-run; `adopt_test.go:864` and `worker_test.go:251` cover only the first run | gap (finding 5) |
| The recorded paused slugs are kept across the re-judge (summary claim) | none asserted | gap (finding 5) |

**Findings**

1. MEDIUM, design. `internal/adoptcli/deps.go`, the `Reconcile` seam: nothing pins that the adopt's reconcile gets `PushLockWait: reconcilePushLockWait`.
   - Dropping the option, which is the fix for the first note's second half, fails no test.
   - The engine test sets the option itself, and the adopt tests use fake reconcile seams.
   - The test should be a hub-level adopt integration row, or an assertion on the seam.
2. LOW, design. `internal/adoptcli/caller.go:72` and `worker.go:55`: the production `ExemptPause` wiring is untested. A nil or wrong function passes every test because each one injects a fake.
3. LOW, design. `internal/fabricengine/originreconcilemoves_integration_test.go`, the new row: no case holds the write lock with a nonzero `PushLockWait`. `waitRecordsLocks` must return busy at once there, and a regression to waiting on any held lock passes.
4. MEDIUM, design. `internal/fabricengine/destroy.go`, `ownedUntrackedTrackedAt` and `untrackedOnPaths`: only the happy path of the destructive removal is covered. This is the rubric's "behavior with no coverage" for code that deletes files. The rows that should exist:
   - an ignored file is kept;
   - a file origin does not track is kept;
   - a code-side untracked file is kept;
   - the gate refuses a file the index tracks;
   - the gate refuses a file the commit does not track;
   - the removal-failure message names the archive tag.
   
   Most fit as rows or assertions in the existing `TestDropLocal` and a gate table.
5. LOW, design. `internal/adoptcli/worker_test.go`: the re-judge row covers only the stop that no longer holds. A re-run whose stop still holds, ending stopped with the `--discard` way forward, is untested. Nothing asserts that the recorded paused slugs survive the clear.

Test economy: the new tests duplicate no existing coverage that I can see. From the diff, `TestRunAdopt`, `TestRunWorker`, `TestDropLocal` and `TestReconcileOrigin_Moves` each gained a row or assertion rather than a new test function, and the redundancy report lists the adopt tests as unique.

**Tool calls**
1. grep for `reconcilePushLockWait`, `PushLockWait`, `StateStopped` and ignore rows in the adopt and fabric tests: to check whether any test pins the adopt's wait option, the stopped-record cases, or an ignored-file case; the manifest lacked the other test files.
