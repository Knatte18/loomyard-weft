```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: re-ran each set-shaped finding's search to check the fix covers every member.

**Findings**

1. **LOW** — `internal/pairteardown/doc.go:60` is stale after the F4 change.
   - It says the gone-worktree Warn names "the slug and that a pull request whose head is its task branch may stay open".
   - `pullrequest.go:81` now words it "from its task branch, if the run opened one", and adds a `branch` attribute naming the branch.
   - The doc and the Warn text now disagree, and the doc doesn't mention the branch.
   - Reword the doc to say the Warn names the slug and the task branch.

2. **No finding (F2 twin, `internal/landingshed/finalize_pr.go:165`).**
   - `dropRecordedPullRequest` also calls `RemoveRecord` before anything stops a session, which is the same shape as the F2 defect.
   - The file's only match for `Session` or `StopStrand` is outside this function, so it never stops the recorded session before the removal.
   - Finalize runs after PR-Review's `Call` has already ended and stopped the session via `endSession`.
   - A session left live by an interrupted earlier PR-Review call would stay orphaned through this path, but the review did not name this site.
   - Treat it as a note for the parent to weigh, not as a defect of this fix.

**Sets that hold**
- **F4:** every caller of `closeOpenPullRequest` passes the new `taskBranch` argument: `pullrequest.go:45`, `pullrequest.go:72` and `teardown_test.go:1272`. The gone-worktree test row asserts `branch=warp/slug`.
- **F2:** `prreview.go:173` is covered by the fix. The `prreview.go:212` site (the `ClosedForContinue` re-entry) needs no stop, because a record in that state has no live session.
- **F3:** `ensureSession` is the only `StartSession` caller, so the stop-on-failed-write covers every start path.
- **F1:** `gateHead` has a single call site, `prreview.go:426`, which now runs after the status read and skips merged states.

**Calls**
1. Bash: grep for `closeOpenPullRequest(` callers, `RemoveRecord` sites, `StartSession` and `ensureSession` sites, `gateHead` and `checkHead` sites, and the gone-warn text. One search per finding was needed, and my context lacked every result.
2. Bash: `sed` and `grep` on `finalize_pr.go`, to see whether the second `RemoveRecord` site stops a session. My context did not hold that file.
