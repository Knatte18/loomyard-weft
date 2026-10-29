# Card 17 — quiet-check-pr-review-wait

**Create:**
- `plan:internal/battencli#childAtWaitingRow` -> `func childAtWaitingRow(taskLocation *lymcwd.Location) bool`

**Delete:**
- `internal/battencli#childAtBeamAwait`

**Edit:**
- `internal/battencli#readAgentActivity`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`
- `internal/battenshed/doc.go`

**Uses:**
- `internal/loomshed#NamePRReview`

**Intent:**
D10: for the whole PR-Review wait the quiet check reads the run as waiting, so the quiet notice does not go off; no new notice is sent and dobby is unchanged.
None of the live waits `readAgentActivity` counts fits the row: its gate's verify marker is live only during a gate, it takes no parent lock, and its session is unawaited, so no shuttle wait marker is written.
`childAtWaitingRow` replaces `childAtBeamAwait`, with the same reads and the same unreadable-status rule: it reports the child's own status reading running at the pawl recipe's `Beam-Await` row or at `loomshed.NamePRReview`, live for as long as the status reads running at either; `readAgentActivity` calls it in place of the old one, and its doc comment names both rows.
`TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` gains rows `RunningAtPRReviewIsAWait` (`{"current_producer":"PR-Review","state":"running"}`, live) and `BlockedAtPRReviewIsNoWait` (`blocked`, not live), and its doc comment names the PR-Review read.
`internal/battenshed/doc.go`'s list of live waits that hold off the quiet notice names a child status running at the PR-Review row.

**ImpactSummary:** One more producer name in the quiet check's child-status read, whose one caller is `readAgentActivity`; every other wait read is unchanged.

**Commit:** 17: the quiet check reads a run at PR-Review as waiting
