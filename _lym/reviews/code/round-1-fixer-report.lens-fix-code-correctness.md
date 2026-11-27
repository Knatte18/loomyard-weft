```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the round-1 change set.

**BLOCKING: `gateRefusalReason` in the ended-review halt (`internal/landingshed/prreview.go`, F6).** Following its way forward pushes the refused head and records it as gated.
- The reason says `run lym loom resume, which gates the task head again before the row pushes it`.
- Take `ReviewEnded: true`, `GatedHead: h1`, task head `h2`, with the gate failing on `h2`. The wake now halts.
- On resume, `openPullRequest` runs `if !found || rec.ReviewEnded { pushTask(...) }`, which pushes `h2` before any gate.
- Then, since `head != rec.GatedHead`, it removes the record and falls through to `findOrOpenPullRequest`.
- `recordPullRequest(ref, head, false)` writes a new record with `GatedHead: h2`, so `gateDue` is false and PR-Review never gates `h2`.
- This holds both when the head is left as it is and when a fix is committed first.
- F6's premise, that an ungated head passes PR-Review, therefore survives one resume. The new row `AGateFailureInAnEndedReviewHaltsAndMailsNothing` stops at the halt and does not see it.
- The entry push predates the fix, but the halt and the way forward that leads into it are the fix's.

**MEDIUM: test row `AFailedWriteOfTheStartedSessionsRecordStopsIt` (`prreview_session_test.go`, F12).** It passes with the behaviour it claims to pin removed.
- `updateRecord` writes the record and only then calls `CommitRecord`, so when the second commit fails the record on disk already names the strand.
- With `ensureSession`'s `StopStrand` call deleted, the error `push stopped` is not transient. `endSession` then calls `stopRecordedSession`, finds the strand live and stops it.
- The row still sees starts 1 and stops 1.
- `ensureSession`'s own stop is observable only when the commit error is transient (so `endSession` spares the session), or by checking state before `endSession` runs.

Everything else I traced gave no finding: `sessionStopWanted` against `judge`, the `endSession` transient exemption, the closed-without-flag session rule, the `pr-continue` admission and its refused rows, and the other new rows.

Tool calls: none; the context held everything this lens needed.
