---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/batchingnudge.go, BatchingNudge and awaitCallInTranscript: the context threshold is now checked on the first tail, before the call's own message is written, and again on the final tail inside the state lock. Settle whether the two readings can disagree in a way that wrongly silences or wrongly waits, and whether the first-tail check at attempt 0 holds for a sidechain transcript."
  - "BatchingNudge: the state file is read outside the lock and the transcript wait runs outside the lock too. Settle whether parallel calls of one message can still nudge at most once, and whether the dropped `current < 0` guard inside the UpdateJSON closure is safe, since `messages[current]` is indexed there."
  - "awaitCallInTranscript: the size-change gate on rereads, the attempt bound, and the stat and read error paths. Settle whether a transcript that is rewritten at the same size, truncated, or unreadable mid-wait leaves the loop correct and bounded."
  - "The decision record's Goal asks to find why the hook does not fire live on edit follows and either fix the gap or remove the hook. Settle whether the diff's cause and fix cover edit follows, including a later edit of a parallel message and the early calls the two-second bound can miss, and whether anything beyond that goal was changed."
  - "_lym/landing/summary.md claims: no wait for a run with no state file, a chain past its cap, or a chain below context_tokens; `New` and `NewFromConfig` both set `transcriptWait`; the package doc describes the wait. Check each against the diff, including any other `Claude` constructor that leaves the zero value and never waits."
  - "Tests in batchingnudge_test.go: whether each new row (late edit fires, late below threshold, late past cap, absent call after the bound) covers behavior no existing row covers, whether wantSleeps on every row is a duplicate assertion, and whether the changed CorruptStateIsReportedAndExitsZero wantStderr in hookbatching_test.go still pins the error path."
  - "Comments and docs as the diff leaves them (batchingnudge.go, claudeengine.go, doc.go): follow the repository's comment conventions and its constraints document, and do not restate perishable specifics such as the tick or attempt values in prose where the constant name suffices."
---

Seed focus for round 1 over `74ea6b01..HEAD`: one behavior change (wait for the call to reach the transcript) with tests and docs.
