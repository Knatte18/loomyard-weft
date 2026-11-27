```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out.

List of findings:

1. **MEDIUM**: F6, `gateRefusalReason` in `internal/landingshed/prreview.go`, as used for an ended review. The way forward the reviewer asked to restore is false there, and following it passes the refused head.
   - The reason says `lym loom resume` "gates the task head again before the row pushes it".
   - On resume, `openPullRequest` runs `pushTask` first, because the condition is `!found || rec.ReviewEnded`. The refused head is pushed before any gate.
   - Since `head != rec.GatedHead`, it then removes the record, finds or opens a pull request, and `recordPullRequest(ref, head, false)` writes `GatedHead: head`.
   - `gateDue` is therefore false and the gate never runs on that head. An approval then ends the row Done.
   - F6's premise, an ungated head passing PR-Review, is only postponed to the resume.
   - The new row `AGateFailureInAnEndedReviewHaltsAndMailsNothing` stops at the halt and never re-enters the row.

2. **MEDIUM**: F11 combined with the gate mail, `openPullRequest`'s `keepsSession` and `gateHead`. A closed pull request with no continue flag now has no session, but the wait still gates before it judges.
   - Case: the record holds the `StrandName` of a session stopped earlier, the pull request is closed on GitHub, and the clean task head is ungated and fails the gate.
   - `gateHead` mails the stopped strand. If the post succeeds, as F6's own premise says it does for a stopped strand, `MailedHead` is set and `judge` then halts "closed".
   - The operator follows that halt (reopen, then resume). `gateDue` is false because `head == rec.MailedHead`.
   - The resumed session is never told, the head is never gated or pushed again, and the review waits silently. This is the defect F5 describes.
   - Before F11, a session was started at entry, so the mail went to a live strand. I did not verify creel's behaviour for a post to a stopped strand. If the post fails instead, the row halts with the gate reason, which is harmless.

No other finding from the proposals for F1, F3, F4, F9, F10, F13–F18.

Tool calls: none.
