```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix covers every member of the sets the findings name (stale "retry once" wording, stop-at-once tests, side wiring).

Findings:

1. LOW — `internal/fabricengine/pushanchored.go:131`: "Each side retries a rejection once, after a fetch, when the remote tip is already contained in local HEAD."
   - It is the same stale sentence as F1 and F2, in a doc comment the review did not name and the fix did not touch.
   - The retry is now looped over the push backoff, as `pushanchored.go:76` and `pushclassified.go:88` say.
   - Evidence: grep for `retries a rejection once|retried once` over the package.
   - Suggested fix: reword it to retry over the push backoff, while origin's tip is contained in the local tip for the code side.

2. LOW — F5, stop-at-once coverage: the destructive-refusal, `ErrPushRetriesExhausted` and transport-after-rejection branches of `rejectionRetryKind` still have no sleep-count assertion.
   - The fix added `sleeps == 0` to the anchor-stop, replay-conflict, auth and never-pushed-transport subtests.
   - The auth and never-pushed-transport subtests (`pushclasses_integration_test.go:382`, `:414`) fail on the first push, not in a rejection retry, so `settleRejection` is never entered.
   - A regression that made those two classes retryable would still pass them.
   - The review named "anchor-stop, replay-conflict, auth and transport" subtests, so this matches its fix key.
   - The premise's own set (the six non-retryable classes) is only partly covered by tests that reach the loop.
   - Evidence: `pushclassified.go:194-201` lists the classes; the grep shows no other `sleeps` assertion for them.
   - Not checked further, since the tool-call cap was in play.

No other stale "retried once" or "never retried" wording about push rejection remains in `internal/fabricengine`, `internal/gitrepo`, `docs`, `PATTERN.md`, `pattern` or `contracts`. The other grep hits are unrelated subsystems.

F4 is covered on all three sides:
- Code: `pushanchored_integration_test.go` asserts the sleeps and one record; the new `pushclasses` row adds the mail.
- Bolt: sleeps and one record are asserted, with no mail because that test runs in parallel.
- Records: the existing test.

Findings left unchecked: none.

Tool calls:
- grep for stale retry-once and never-retried wording across Go and markdown files: the context only held the three files the review named.
- grep for rejection-retry descriptions in `fabricengine`, `gitrepo`, `docs`, `PATTERN.md`, `pattern` and `contracts`: it checks sites outside the named files.
