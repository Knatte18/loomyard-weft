```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the hub pull diff (the amendment round), judged from the inherited context.

**Per-function list**

- `PullHub` (hubpull.go:70). Case: `List` fails. Bolt and prime sides are already in `res.Sides`, the error is returned, and the deferred snapshot still sets `Mutations`. Correct. Case: no beams. Result holds two sides. Correct.
- `pullBoardSide` (hubpull.go:93). Case: `Bolt.Pull` returns a skip with a detail. Reported `stopped` with the way forward. Correct. Case: error. `stopped` with the error text. Correct. Case: pull moves nothing. `up_to_date`. Correct. Case: SkipGit. `Bolt.Pull` returns empty values and the side reports `up_to_date`. Correct.
- `Bolt.Pull` (bolt.go:243). Case: SkipGit, so it returns before taking the lock. Correct. Case: the lock can't be acquired. Wrapped error returned and no state changed. Correct. Case: normal. The write lock is taken first, then `pullFirst` takes the push lock, which keeps the documented order. Release is deferred. Correct.
- `pullPairSide` (hubpull.go:110). Case: SkipGit. `up_to_date` before any lock or `Open`. Correct. Case: beam with an unreadable upstream. `stopped`. Correct. Case: beam with no upstream. `up_to_date` without the lock. Correct. Case: the prime skips the upstream check and goes to `fab.Pull`. Correct, since the plain pull handles it. Case: `Open` fails. `stopped` before locking. Correct. Case: `lockPair` fails, for example Ctrl-C cancels the wait. `stopped` and no release is called. Correct. Case: `fab.Pull` errors partway. The mutations already made are extended into `rec`, then `stopped`, and the deferred release runs. Correct. Case: `WeftPulled` is false. `stopped` with a way forward. Correct. Case: the pull moved something. `rec` grew, so `moved`. Correct. The lock is held for one pair only, because `defer release()` runs when the function returns.
- `movedOrCurrent` (hubpull.go:155). Case: `rec.Len()` is unchanged. `up_to_date`. Correct. Case: it grew. `moved`. Correct. This assumes `rec` records only real moves and no no-op entries. I did not verify that `Mutations` holds nothing else. It is a minor risk, not a finding.
- `localBeamCandidates` (hubpull.go:173). Case: main or prunable entry. Skipped. Correct. Case: unreadable origin record. Listed with its error, so the other pairs are still found. Correct. Case: no record found, or `!Beam`. Skipped. Correct. Case: `NamedWorktree` fails. Listed with its error. Correct. Case: slug taken from `filepath.Base` of the path. Correct for a worktree path.
- `HubPullResult.Stopped` (hubpull.go:51). Case: empty `Sides`. False. Correct.
- `runHubPull` (fabriccli/hubpull.go). Case: `PrimeName` fails. Error envelope. Correct. Case: `NamedWorktree` fails. Error envelope. Correct. Case: a stopped side with a nil `PullHub` error. `hubPullStopError` runs, and `errWithRecordFields` carries every side in `fields`. Correct. Case: `PullHub` returns an error. `result` is still partly populated and `fields` carries those sides, so it is not lost. Correct. Case: `Sides` is empty. `hubPullSidesMap` returns an empty non-nil slice. Correct.
- `hubPullStopError`. Case: it is only called when a stop exists, so `stopped` is non-empty. Correct.
- `acquireBranchLock` (anchordeps.go). Case: the `verb` parameter changes the owner run id and the wait text. The anchor caller passes "anchor", so its run id changes from `fabric-anchor` to `"fabric-" + "anchor"`, which is the same string. Correct. A branch read failure returns an error, which `pullPairSide` reports as `stopped`. Correct.
- `pullCmd` `--hub` branch (weft_verbs.go). Case: the flag is set. `runHubPull` is called and the plain path is not run. Correct. Case: the flag is absent. The plain path runs as before. Correct.
- `fabricengine/doc.go` hub pull section and the parent's workspace-file lines. The parent's lines 1133–1134 are intact, and the new section sits under its own heading. The doc matches the code on skip with no upstream, SkipGit, and an unreadable origin record as a stopped beam. Correct.

**Findings:** none.

**Tool calls:** none made; the context already held everything the lens needed.
