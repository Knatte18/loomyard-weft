# Webster-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit per finding, none pushed.

## Fixed

- **F1 (BLOCKING), `1205cacf4`.**
  `discussionCommitPathspec` now takes the location and adds `LoomParentReviewDirRel()` only when `holdsFile` finds a file under the parent-review directory, as `statusCommitPathspec` does.
  A run with no reviewer, or with the entry off, no longer fails its discussion commit on git's pathspec refusal.
  `TestDiscussionCommitPathspec_IncludesParentReviewDir` covers three cases: the directory absent, holding only an empty round, and holding a request file.
- **F2 (BLOCKING), `0161653f4`.**
  `resolvePersistentPreRun` now returns early for the review group and every command under it, through a new `inReviewGroup` helper.
  Its doc comment names the reason, `cmd/lyx`'s `cobra.EnableTraverseRunHooks`, and review.go's file comment now says the parent's pre-run skips the group.
  The new test `TestResolvePersistentPreRun_SkipsReviewGroup` calls the loom parent's pre-run for the group and each verb over a non-git cwd and asserts it writes nothing and does not abort.
  With the skip disabled, the test fails for all five commands.
  A rebuilt dev `lyx` now prints one envelope per refusal, the review verb's own.
- **F3 (MEDIUM), `9190ab11d`.**
  `discussionCommitPathspec` and `newParentReviewConfig` now sit above `wire`'s doc comment, so the comment documents `wire` again, and wiring.go is gofmt-clean.
  `gofmt -l` over every Go file in the diff reports nothing.
- **F4 (LOW), `884d19b50`.**
  `RecordVerdict` now re-reads `request.json` under the verdict's own round lock, refusing `ErrExpired` or `ErrNoOpenRequest` when the request changed since its first check.
  It writes `review.md` inside that same critical section, after the verdict-found check.
  `MarkExpired` now refuses with `ErrVerdictRecorded` when `verdict.json` exists, checked under the request's round lock.
  On that refusal the gate and `final` re-run their own evaluation, so the verdict is read rather than expired.
  Both held-lock reads go through a new `readHeldJSON`, because `state.ReadJSON` would re-acquire the held lock.
  New tests: `TestRecordVerdict_ExpiredSinceRead`, `TestMarkExpired_RefusesRecordedVerdict` and `TestGate_VerdictLandingBeforeExpiryIsRead`.
  The race tests use the store's injected clock, which runs between the read and the write, to stand in for the concurrent writer.
- **F5 (LOW), `8f91a9bfd`.**
  Each comment F5 lists now uses semantic line breaks.
  `runStatusWatch`'s comment was rewritten too, since it still called the unavailable line "precomputed" after the diff moved that line into `statusWatchLine`.
- **F6 (NIT), `0aaccbfc9`.**
  The comment now names `hasRound`.

## Verification

`go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` were run after the last fix; results are in the closing summary of this round.

## Deferred

None.
