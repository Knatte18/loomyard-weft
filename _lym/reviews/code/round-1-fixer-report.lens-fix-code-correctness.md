```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I walked each changed branch with a concrete input, and the callers and added tests hold. No findings.

- **`RecordScope` and `dropCaptureSkipping`** (the F1/F3 change):
  - Capture skipped `[a]`, then a replacement to `off`, then to `full`: `off` leaves the capture in place, and `full` finds `a` in `capture.Skip` but not in the new skip list, so it removes the capture.
  - A removal or read failure inside the update callback returns an error from `UpdateScope`'s callback, which writes nothing, so the old scope stays and a retry works.
  - The `!found`, agent and already-settled branches return before the drop, as before.
- **`RunCapture`** writes `Skip: req.Skip` into the running record, and the later writes reuse the same `rec`. Both RunCapture call sites pass `scope.Skip`: the production child and the in-process fallback.
- **`childNoRecordError`** (F5):
  - The only producers are the two `startFailure` calls in `awaitRunningRecord`, and `StartCapture` returns that error unwrapped.
  - `errors.As` with a value-type target matches it.
  - Other `StartCapture` errors (held lock, log directory or file, a child that fails to start, a refused breakaway) are no longer matched, so they fall to the error path or `ErrBreakawayRefused`, as intended.
  - The driftcli caller (`cli.go:345`) only prints the error text, which is unchanged.
  - The `loomcli` seam and the `envkit` stub are unaffected.
- **Tests that fail with the defect in place:**
  - The chain subtest fails against the old code: `off` set `cur.Skip` to nil, so `full` kept the capture.
  - The unremovable-capture subtest fails against the old code, which wrote the scope before the removal failed.
  - The environment-fault baseline row fails against the old code, which turned any non-breakaway error into a Stuck.
  - The `capture_test.go` `Skip` assertion fails if `RunCapture` stops recording `Skip`.
- **The F2 integration tests:**
  - They fail if `compare`, `capture --side` or `RunCapture` reverts to the worktree root, because `pwd` then differs from the anchored path.
  - They also fail if the capture child's start directory reverts, because the `lymcwd.Resolve` cwd gate keeps the child from recording its start.
  - Not pinned: the `loomcli` `StartCapture` seam (`driftwiring.go:143`). It calls `os.Executable`, so a test cannot run it. Reverting it to `WorktreePath()` fails no `loomcli` test. The driftcli test covers the equivalent `startMainCapture` call site but not this one.

Tool calls made:
1. `grep -rn "StartCapture|startFailure|childNoRecordError"` over non-test Go files, to confirm all callers of the changed error type and `StartCapture`. My context had only the files I had read.
