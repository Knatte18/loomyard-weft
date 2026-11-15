```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the `fix-code-correctness` lens over the F1–F10 fix set.

No findings.

I ran concrete inputs through each changed branch:
- **`readEdits` with a new untracked file:** `Changes()` returns `["new.go"]` against a spawn record of nil. `samePaths` is false, so the problem is "uncommitted changes" and the resume becomes a precondition hand-on. With the compare removed, the `a resume over a new untracked file hands on` row would resume, so it fails with the defect in place.
- **`readEdits` with a reordered path list:** `["a.go","b.go"]` against `["b.go","a.go"]` compares equal, because `samePaths` sorts clones. The `a resume leaving an untracked file it found goes on` row covers this.
- **`readEdits` with a failing `Changes()`:** the call returns the commits plus a problem, and no caller dereferences a nil result. Only `shedarbiter/call_test.go` and `loomcli/arbiter.go` build a `WorktreeSeam`, and both set `Changes`.
- **`readEdits` when `IsAncestor` or `CommitLog` fails:** the problem names `HEAD moved from aaa to bbb`. `conclude` now passes the whole `edits` to `handOnWith`, so the question carries the "Check the worktree first" note and the commits. The earlier gate failure confirmed the old `attemptEdits{commits: ...}` call hid the problem, and the rerun passed.
- **`settleFresh`:** a `Changes()` error is refused with the same text as a `Snapshot()` error. `attempt.Changes` is recorded, and the new `attempt changes` assertion pins it.
- **`TestCall_Edits` `viaSeam` row:** the second call through `Seam(...).Judge` takes `Settle` → `settleExisting` → `conclude`. It spawns nothing and returns the same envelope. If `Judge` dropped `Commits`, the assertion would fail.
- **`TestArbiterEnvelopeFields`:** a nil `commits` key renders as a non-nil `[]string{}`. A dropped key or a nil slice fails the `got == nil` check.

Not a finding: an arbiter that commits a path that was already dirty at spawn still reads as "uncommitted changes". `Dirty` already behaved this way before the fix, and the `Changes` comparison only adds untracked files to it.

Tool calls: none; my context already held every changed file.
