```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I found no defects in the code the fix changed, and one LOW finding on two of the added test assertions. The doc-comment edits (F1–F3) are text only and nothing in them can break a caller.

I did not re-run the tier1 gate. I take the parent's green run (`lym gate test --tier 1 ./internal/fabricengine`) as covering all of these edits.

**Walked through step by step**

- **Code-side persisting rejection.** This is the new `ACodeRetryWhoseEveryPushIsRejectedWritesANotFastForwardRecordAndOneMail` row, and the changed step in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`.
  - The always-declining hook rejects the first push.
  - `repushOverOriginContained` fetches and finds `HasUnpulled` false, so its push is rejected again with a bare rejection.
  - `rejectionRetryKind` returns `retryable`, so `settleRejection` sleeps once per delay and stops at `attempt == len(delays)`. That gives `sleeps >= len(PushBackoffDelays())`, one `not_fast_forward` record and one mail.
  - If the defect returned (one retry, no loop), `sleeps` would be 0 and both new assertions would fail, so they do catch it.
- **Bolt persisting rejection.** The bolt case asserts `sleeps >= len(delays)` and one `bolt` record.
  - The replay retry returns a bare rejection, so the loop runs the whole backoff and a one-shot retry would fail the assertion.
  - The record side is `filepath.Dir(b.path)`, the same value as the `f.hub.Path` passed to `requireOneFailedPush`.
- **Replay-conflict and anchor-stop rows.** Both are non-retryable by `rejectionRetryKind` (`ErrReplayConflict` and `AnchorStopError`), so `sleeps != 0` is meaningful. Without it, a regression that retried them would pass.

**Findings**

- **LOW** — the auth row of `RepeatedAuthRefusalsShareOneRecordAndOneMailAndALandedPushRemovesIt` and the `OfABranchNeverPushed` transport row, both with the new `sleeps != 0` assertion.
  - Both pass with the defect in place.
  - `PushRebaseFree` returns `ErrPushAuthRefused` directly from the first push (`internal/gitrepo/push.go:146-160`, "never maps to ErrPushRejected"). So `pushClassified` never enters `settleRejection` or `rejectionRetryKind`.
  - A never-pushed branch has no backoff either.
  - `sleeps == 0` therefore holds whatever `rejectionRetryKind` says about `ErrPushAuthRefused` or transport failure, and those two lines pin nothing.
  - They are harmless. To pin the review's F5 premise for those classes, the push would have to be rejected first and have the retry return an auth refusal or a transport failure.

**Tool calls**
1. `grep ErrPushAuthRefused internal/gitrepo/*.go`: I needed to know whether a hook-declined auth push is classified as auth refused on the first push or only after a rejection; my context did not show `gitrepo/push.go`.
