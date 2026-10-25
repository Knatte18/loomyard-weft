```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F1–F4 as the fixer carried them out, under the fix-proposal lens.

**Findings**

1. **LOW**: `CompareRequest.LockRetry` in `internal/driftengine/compare.go`.
   - **Summary:** A zero `LockRetry` now makes `RunCompare` refuse even a free marker, and the field comment does not say so.
   - **Evidence:**
     - `AcquireWriteLockWithin(path, 0)` builds an already-expired context. The fixer's first test run showed it: with the fixture's `LockRetry` unset, every `TestRunCompare` row failed with "a compare is already running".
     - Before the fix, a zero-valued `CompareRequest` worked.
     - The two production callers, `internal/driftcli/cli.go` and `internal/loomcli/driftwiring.go`, both set it, so nothing ships broken.
     - A future caller that leaves it unset gets a false "already running" refusal, which is the failure F1 was meant to remove.
   - **Fix to consider:** state in the field comment that it must be positive. A fallback to a non-blocking try when it is zero would also do. `CaptureRequest.LockRetry` has the same trap.

**Proposals with no case against them**
- **F1:** the retry is bounded and probes release at once. A real second compare now waits `LockRetry` before it refuses, which is acceptable.
- **F2:** comment only. The reworded text matches `internal/driftengine/verbs.go` (a ruling is set only under scope full: unruled for the agent, ruled for a caller) and the rule in `internal/driftengine/verify.go` that rules an unruled acceptance the halt names.
- **F3:**
  - `killCapture` sits after the owned-process end, which has already ended a registered loop, so the call is safe while a loop is live.
  - It is also safe with the task worktree gone, since `EndSession` calls it in that state too.
  - A failure stops the cancel before strands are stopped, and the error names the retry command.
- **F4:**
  - `Check` returns the HEAD it verified under the lock, and `RunCapture` records that value.
  - The window between `Check` and the command starting still exists, as it did before. Drift-Await's `capture.Head != head` comparison still catches a moved HEAD, so this is not a regression.
  - The `rec.Head == "head1"` assertion in `internal/driftengine/capture_test.go` now reads `Check`'s return, since the `Head` field is gone.

**Tool calls:** none. My context already held every file I needed.
