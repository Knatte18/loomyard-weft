```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (`prreview.go`, `prreview_wait_test.go`, `_lym/landing/summary.md`) for agreement with its surroundings.

**Findings**

1. **LOW** — `_lym/landing/summary.md`, "Tests" section. It says `TestPRReview_Wait` "gains two rows".
   - The change set adds a third row, `AFailedLivenessReadKeepsWaitingAndLogsOnce`, and the summary was edited in this round without updating the section.
   - The section also still describes only the old shape of `ASessionThatEndsWhileTheReviewIsOpenHaltsNamingTheResume`, which now halts after judging a polled status, not on a no-status wake.

2. **LOW** — `_lym/landing/summary.md`, line 8: "each wake that leaves the review open reads whether the session the call keeps is live".
   - After F1 only a wake that read the status and left the review open reads liveness. A wake that read no status returns nil first.
   - The "Where the check runs" bullet was updated but this intro sentence was not.
   - It matches the new `wake` doc comment: "A wake that reads the status and leaves the review open …".

3. **LOW** — `_lym/landing/summary.md`, "Only a session the call keeps is watched": "A record whose review has ended is never watched either."
   - F5 removed the `rec.ReviewEnded` guard from `haltOnEndedSession`, so the check now depends on `keepsSession` alone.
   - The sentence still holds in practice, because `endReview` ends the call in the same wake. The mechanism it implies, a record check, no longer exists.
   - It should say that a wait entered on an ended review passes `keepsSession=false`.

No other disagreement found. These all agree with the new code and tests:
- the `wait` and `wake` doc comments
- the `prWaitState.liveWarned` comment
- the `haltOnEndedSession` doc, which now states the Warn-once behaviour
- the `AReviewEndedAtEntry…` row, which now pins `keepsSession=false`
- the Warn text, which matches the test's count string

Further tool calls: none. My context already held every changed file.
