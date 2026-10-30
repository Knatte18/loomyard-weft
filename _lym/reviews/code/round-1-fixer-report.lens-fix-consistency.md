```yaml
lens: fix-consistency
outcome: reported
```

Findings of lens fix-consistency:

1. **LOW**, `internal/pairteardown/doc.go:60`.
   - Summary: the package doc still describes the gone-worktree Warn as naming only the slug, and it says "a pull request whose head is its task branch may stay open".
   - Evidence: `internal/pairteardown/pullrequest.go:81` now logs a `branch` attribute and the wording "a pull request from its task branch, if the run opened one, may stay open". The `closeOpenPullRequest` doc comment was updated, but this twin was not.

2. **LOW**, `internal/landingshed/prreview.go:371` (the `wait` doc) and `docs/overview.md:580`.
   - Summary: both read the old order, "gates and pushes … reads the status … and judges it".
   - Evidence: `wake` now reads the status first and skips the gate for a merged pull request. Neither text says a merged pull request is never gated or pushed, and this is the behaviour finding F1 added. The `wake` code comment states it, but the two docs do not.

3. **LOW**, `internal/landingshed/prreview.go:208-212` (`openPullRequest`, the `ClosedForContinue` re-entry at a head past the gated one).
   - Summary: this path still calls `RemoveRecord` without first stopping the recorded session, which is the defect F2 fixed in `closeRecordedPullRequest`.
   - Evidence: the new record is written without `StrandGUID`, so a strand still live from an interrupted earlier call is orphaned, and `ensureSession` starts a second one. The window is narrow, because `endSession` normally stops the session on every verdict.

4. **LOW**, `internal/pairteardown/pullrequest.go:37-43`, the new gone-path config load.
   - Summary: a failed `fabricengine.LoadConfig` now returns early with a different warning.
   - Evidence: the `closePairPullRequest` doc says it "reads whether the task worktree is gone, and when present resolves…", and does not say the gone path loads the fabric config. `TestCloseOpenPullRequest` calls `closeOpenPullRequest` directly, so the new config-failure branch has no row. This is minor, since the branch fails safe.

Tool calls:
1. `grep` over `internal` and `docs` for the old Warn wording and the gate-then-poll phrasing, because my context held neither `doc.go` nor `docs/overview.md`.
