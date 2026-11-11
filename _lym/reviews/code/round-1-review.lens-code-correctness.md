```yaml
lens: code-correctness
outcome: reported
```

Task: judge the correctness of the changed code by running concrete cases.

**Per function, case run, verdict**
- `runAdopt` and `runWorker` exemption: both call `defer deps.ExemptPause()()` first. `replaceGateExemptAll` returns false and sets nothing when no gate is installed. `tendercli/gate.go:61-90` installs the gate and returns silently on a resolve failure, in which case no pause gate holds the process either. Correct.
- `waitRecordsLocks` with `PushLockWait = 0`: one try, then it returns, so a held push lock reads busy at once. Correct.
- `waitRecordsLocks` with `PushLockWait = 3s` and the lock held throughout: tries at waited 0, 1, 2 and 3, which is 3 sleeps, then busy. It matches the test's `wantSleeps` 3. Correct.
- `waitRecordsLocks` with the write lock held: `tryRecordsLocks` returns `pushHeld` false, so it returns busy without waiting. Correct.
- `waitRecordsLocks` with a try error: it returns at once. Correct.
- `waitRecordsLocks` bound: only the sleeps count toward the wait, not the time the tries take. The wait is still finite. Correct.
- `ReconcileOptions.sleepFor`: the retry path still sleeps `RetryInterval`. Correct.
- `reconcileRecordsBranch` and `originMover` now carry `opts` through. Correct.
- `untrackedOnPaths` / `DropLocal`, an untracked file origin added: it appears in the diff of local HEAD against origin's tip and in `ls-files --others`, so it is removed after the tag and before `reset --keep`. Correct.
- `untrackedOnPaths`, an ignored file: it is excluded by `--exclude-standard`, and git overwrites it. Correct.
- `untrackedOnPaths`, a code branch: it is skipped by the `side == failedPushSideRecords` guard. Correct.
- `untrackedOnPaths`, a nested path: `ls-files --others` lists files individually, so it matches the slash paths of the diff. Correct.
- `ownedUntrackedTrackedAt` / `resolvePathOwnership`, an index-tracked target: it is refused. A target that origin's tip does not track fails `FileAtRevision` and is refused, so an operator-authored file is never removed. Correct.
- `ownedUntrackedTrackedAt`, `plan.onto` as the revision: it is a tracking ref, not a SHA. `FileAtRevision` checks `validSHA(rev)` and returns `ErrInvalidSHA` otherwise. I did not resolve whether `target.tracking` is a 40-hex SHA or a ref name. If it is a ref name, every removal is refused and the new integration test would fail. Unverified, see the finding below.
- `DropLocal` removal failure after the tag: the tag is kept and the error names it, so the state is partial and recoverable. Correct.
- `rejudges` / `runWorker` with a record stopped at `done`: `restart` becomes true, `Steps` and `StoppedPart` are cleared, and `from` stays at reconcile. `Inputs` is kept, and the test shows `HeldBack` emptied after the re-reconcile. Correct.
- `rejudges` with a record stopped elsewhere, or a `done` record: `rejudges` is false, so it resumes as before. Correct.
- `rejudges` re-run side effects: owner, deploy and beam steps run again for slugs the first run already finished. This is the same restart a changed `--keep-local` makes, so I take it as existing behaviour. I did not check the step idempotence myself.

**Findings**
1. Severity LOW, class `design`, `internal/fabricengine/destroy.go:576-594` (resolution of `pathOwnershipUntrackedTrackedAt`), with `internal/gitrepo/blobread.go:30-33` as evidence. `FileAtRevision` returns `ErrInvalidSHA` for anything that is not a SHA, and `DropLocal` passes `plan.onto` (`target.tracking`). I did not read the definition of `target.tracking`. The ownership predicate works only if that value is a SHA. The new test in `droplocal_integration_test.go` would fail if it were a ref name, so I rate this unverified rather than confirmed. The handler should confirm.
2. Severity LOW, class `design`, `internal/fabricengine/destroy.go:2063-2070`. A path in the local-to-origin diff that origin deleted and that is also untracked locally makes the new gate refuse. `DropLocal` then fails after the archive tag exists, leaving a partial move, although the file is not one origin's tip tracks. The case is rare and the failure is safe: nothing is deleted wrongly.
3. Severity LOW, class `design`, `internal/fabricengine/destroy.go:2108-2110`. `PathTracked` runs `ls-files -- relPath` as a git pathspec, so a file name with glob characters could match other tracked paths. The effect is a false refusal, which is safe.

**Tool calls**
1. Read `originreconcilemoves.go` lines 70-115 and the `adoptcli` `StepDone` uses: the context lacked `tryRecordsLocks` and where the done step stops.
2. Read `tendercli/gate.go`, `steprun.go` and `untrackedOnPaths` / `pathsChangedBetween`: the context lacked where the gate is installed, how the steps loop saves state, and the full text of those two helpers.
