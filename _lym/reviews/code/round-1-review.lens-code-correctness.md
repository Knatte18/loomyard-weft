```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function and branch the diff changes, I ran one concrete case through the code.

**Per function or branch: the case I ran, and whether it is correct**

- `statuscommit.New` (`internal/statuscommit/statuscommit.go:96-128`). The nil-`MergeActive` default is gone, so `deps.MergeActive()` is now called unguarded. Case: the only two production callers, `loomcli/wiring.go:344` and `battencli/commitstatus.go:214`, both pass a non-nil `MergeActive`. `TestNew_NilPushPanics` passes one too. Correct.
- `statuscommit.New`, path where `MergeActive` returns (false, nil) and then `Commit` errors. The call goes to `failureDisposition`, which re-probes: false returns `commitErr` (halt), true returns nil (skip), and a probe error returns nil. The admission is released before the disposition. Correct.
- `statuscommit.New` push path, with `Push` returning an error and `owed` nil. `owed` defaults to false, so the error is returned with no sleep, and the admission is released once. With `pushErr` nil it returns nil. Correct.
- `statuscommit.failureDisposition`, now one return value. The re-probe error, re-probe true and re-probe false cases map to nil, nil and `commitErr`. Correct.
- `loomcli.newCommitStatusSeam` and `battencli.newCommitStatusSeam`: the core is built inline with no layout switch. In batten, `afterCommit` still writes the marker. Correct.
- `fabricengine.rejectionRetryKind` (`pushclassified.go`), case: an `ErrPushRejected` whose text carries "GH006: Protected branch update failed". `ProtectedBranchRefusal` matches "protected branch", so it returns named=true and retryable=false: no retry, class `other`. A bare rejection without the marker still falls through to (true, true). `err` is never nil at this call. Correct.
- `fabricengine.ProtectedBranchRefusal`: lower-cases the error text and checks the two markers. Case: a non-protected error returns false. Correct.
- `landingshed.Finalize.landedDiffEmpty`, cases:
  - empty `parentHeadMerged`: false;
  - nil `ChangedPaths`: false;
  - `ChangedPaths` error: logs a Warn and returns false;
  - zero paths: true;
  - one path: false.
  In a two-repository hub the code repository holds no records path, so a parent records commit cannot hide an empty landing. Correct.
- `landingshed.Finalize.parkedMergeWayForward(checkout)`: one signature now, naming both `--continue` and `--abort`. The read-error branch and the `MidMergeParked` branch both call it. Correct.
- `landingshed.preMerge.clear`:
  - `MidMergeParked` with another source returns the reason with `mergeWayForward`;
  - its own merge-in falls through to the abort-and-redo;
  - a foreign state goes to the default branch.
  Correct.
- `loomcli.ownMergeInLeftover` and `refuseOverUnfinishedMerge`:
  - `MidMergeParked` merge-in of the parent at Finalize is accepted;
  - any other parked merge gets the fabric message;
  - a foreign state goes to the default branch with the git message.
  Correct.
- `lymcwd.ResolveWithin` and `ownerOfRecordsSibling`:
  - cwd in `&lt;hub&gt;/task-&lt;suffix&gt;` with `&lt;hub&gt;/task` present gives owner `task` and `inRecordsSibling` true;
  - a suffixed name with no owner directory comes back unchanged.
  Correct.
- `lymcwd.readRecordedAnchor`:
  - an own marker holding "../out" is an `ErrInvalidAnchor` naming the sibling file;
  - an empty own marker falls through to the Bolt marker;
  - with neither present it returns ("", false, nil).
  Correct.
- `runfsm.Act` and `Verdict.Refusal` (push-owed):
  - a set worktree names `lym fabric push` in it;
  - an empty worktree gives the bare "once the owed push lands".
  `Judge`'s aside clears error and worktree. Correct.
- `pairteardown.runReader.read`: one owed record sets `PushOwed`, its error and its `RepoPath`. Correct.
- `gitrepo.MergeStart`:
  - ref "-x" is refused as `ErrInvalidSHA` before git runs;
  - squash false runs `merge --ff --no-commit`.
  Correct.
- `fabriccli.CloneAndWire`: `PushRecorded` now runs unconditionally, and its error returns an empty `CloneResult`, which the deferred handler covers. Correct.
- `fabriccli.runCloneWithReset`:
  - zero args returns the usage text without `--single-repo`;
  - two args set `warpURL`.
  Correct.
- `fabricengine.Fabric.Commit`, refusal order:
  - a merge record present returns `*ErrMergeInProgress` before the layout read;
  - no record but a foreign state returns `*ErrForeignMergeState`.
  This is byte-identical to f06acf39^. Correct.
- `fabricengine.MergeStateActive`: a weft cherry-pick with a clean index and no `MERGE_HEAD` reports false; `MERGE_HEAD` or a conflicted index reports true. Correct per D7.

**Findings**

None.

A note outside this lens, not filed. With D7's merge-only probe, a crashed replay's leftover `CHERRY_PICK_HEAD` in the weft lets the status commit run. git refuses a pathspec commit during a cherry-pick, and the re-probe reads false, so the transition halts with git's error instead of skipping. That is loud, and it is the pre-f06acf39 behaviour D7 asks for (`internal/statuscommit/statuscommit.go:108-111`).

**Tool calls**

1. Grepped the `statuscommit.New` callers and read `statuscommit.go:80-140`. My context had the diff hunks but not the post-change body, and not whether every caller still sets `MergeActive`.
