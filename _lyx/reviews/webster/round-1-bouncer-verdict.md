---
verdict: BLOCKING
rationale: "F1, F2, F3 and F5 are fixed and verified (new integration tests pass), but F4 is incomplete: comments added by cards 6 and 7 at internal/reedengine/overlay.go:250-251 and internal/reedengine/state.go:6-7 still break mid-sentence at a fixed column, against CONSTRAINTS.md's Comment Line-Break Convention"
---

The review range is right: `_lyx/shed/pair-lifecycle/status.json` gives `product.parent = main`, and `_lyx/loom/rework/` does not exist, so the subject is the whole branch over `git merge-base main HEAD`.
The report has no `## Focus departures` section, so there is no departure to ratify or reject.

All five round-1 findings are valid against the diff and the cards.

- **F1 (BLOCKING), resolved by `88fc1fa4f`.**
  `EndSession` runs the refusal probe before ending a session, so on a fully gone pair it returns `ErrPairNotFound`.
  The `Shutdown` closure in `internal/battencli/wire.go` now logs that error and returns `"", nil`, so `Remove` runs and `finishRemoval` reports done.
  `TestBattenIntegration_Teardown_ReEntryAfterCompletedRemovalIsDone` runs both closures twice and passes.
  The `Teardown:` comment, `internal/battenshed/doc.go` and `manifest/designs/pair-teardown.md` now describe the re-entry path correctly.
- **F2 (LOW), resolved by `2c91423c9`.**
  `commitPendingRecords` returns before `recordTrailerSHA` when no record path is present, and `TestCommitPendingRecords_NothingPresentCommitsNothingWithoutGit` passes.
- **F3 (LOW), resolved by `beee8debc`.**
  `RemoveResult.RemoteSkippedReason` now names both causes, the no-origin case with remote and the origin-only sibling branch kept without remote.
  Its doc matches the two setters, `weftwiring.go:257` and `remove.go:296`.
- **F5 (NIT), resolved by `8d728b786`.**
  `SessionResult.Ended`'s doc now states what each arm reports.
- **F4 (LOW), still open.**
  `f6551f834` reflows every site the review listed and several more.
  Two new comments are still fixed-column wrapped, though, and the review missed both:
  - `internal/reedengine/overlay.go:250-251`, in `hasSession`, added by card 6 (`428408dba`): "lets a test script tmux's exit-1 answer / through the exec seam" breaks mid-sentence.
  - `internal/reedengine/state.go:6-7`, the file-header sentence about `Retiring`, added by card 7 (`51198eb38`): "set by a reed verb / (MarkRetiring), never written" breaks mid-sentence.

  The convention covers new and changed comments, both comments are new on this branch, and neither break falls at a clause boundary.
  The fix is to put each sentence on one line.

`go build ./...`, `go vet ./...` and `go vet -tags integration ./...` pass at HEAD, and so do the unit tests of `pairteardown`, `battenshed`, `fabricengine` and `battencli`.
