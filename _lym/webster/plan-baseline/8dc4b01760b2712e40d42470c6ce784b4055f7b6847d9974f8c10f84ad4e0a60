# Card 11 — pr-review-ending-autonomy

**Edit:**
- `internal/landingshed#PRReview.wake`
- `internal/landingshed#PRReview.judge`
- `internal/landingshed#gateRetryHint`
- `internal/landingshed#Finalize.landingForDecision`
- `internal/landingshed#TestPRReview_Wait`
- `internal/landingshed#TestFinalize_PullRequest`
- `docs/overview.md`

**Uses:**
- `plan:internal/landingshed#PRReviewStanding`
- `plan:internal/landingshed#PRReviewSatisfied`
- `internal/landingshed#PRReview.endReview`
- `internal/landingshed#waitWorld`
- `internal/landingshed#newWaitWorld`
- `internal/pullrequest#Record`
- `internal/landingshed/doc.go`

**Intent:** D4: under a recorded autonomy of true, only the parent's recorded satisfied answer at the gated head ends the review on the continue flag, and a closed pull request always halts.

`judge`, with `rec.Autonomy` true:

- Open, `rec.Continue` set, clean and gated: it reads `PRReviewStanding(head)`.
  `PRReviewSatisfied` ends the review through `endReview`.
  Anything else clears `Continue` through `updateRecord`, hands `wake` a mail to the session that the review stays open at `head`,
  and returns nil, so the wait goes on.
  The mail names waiting for the parent's answer with `lym loom parent-review await <slug>` when the standing is `PRReviewAwaiting`,
  and asking the parent again with `lym loom parent-review ask <slug>` otherwise.
- `wake` stopped the session before `judge` (the continue flag on a clean tree at the gated head), and `ensureSession` may give the resumed session a new strand name.
  So `wake` sends the stays-open mail through `MailAgent` after `resumeSession`, or after `judge` when it stopped no session,
  to the `StrandName` the record holds at that moment, never to the name `judge` read.
  So a commit made after a passing `pr-continue` is gated and waits for a new ask at the new head.
- Closed: `ReviewEnded` with a clean tree is Done and a dirty one halts, as today;
  the `Continue` branches are skipped, so the row never records `ReviewEnded` there and takes today's closed-without-merge halt, which the orch relays as a D5 class 4 severe halt.
- Merged, and an approval of the gated head on GitHub, end the review as today with no parent answer.

With the recorded value false or absent, `judge` runs as today.

`gateRetryHint` drops "and the operator has asked you to continue":
when the failure lies outside the change, such as a file another process held open, the agent runs `lym loom pr-continue`, which gates this head again.

`landingForDecision`, on a closed pull request whose record holds autonomy true, takes its closed-without-merge halt whatever `ReviewEnded` and `decision.MergeAtOnce` say,
and neither reopens nor merges, so a pull request closed on GitHub after the review ended halts Finalize too.

`docs/overview.md`'s PR-Review paragraph, which says the row ends Done when the operator's `lym loom pr-continue` ends the review and words the gate-failure hint,
says that under a recorded autonomy only the parent's satisfied answer at the gated head ends it on the continue flag, and takes the new hint.

Tests, tier0. `waitWorld` holds no standing, so each new row sets `w.deps.Landing.PullRequest.PRReviewStanding` to a closure that answers the row's standing and counts its reads.
`TestPRReview_Wait` gains rows with the record holding autonomy true:

- continue set with the standing satisfied at the gated head ends Done with `ReviewEnded`;
- continue set with the standing `ask-again` clears `Continue`, mails the session a line naming `lym loom parent-review ask`, and keeps waiting;
- continue set with the standing `ask-again` on a live session that the resume restarts under a new strand name: the mail names the resumed session's strand, not the stopped one's;
- continue set with the standing `awaiting` clears `Continue` and mails a line naming `lym loom parent-review await`;
- a commit after the review was continued: the new head is gated, and with the standing `ask-again` at it the review stays open;
- a mailed head cleared by `pr-continue` at an ungated clean head: the next wake gates that head;
- a closed pull request with continue set halts naming "closed on GitHub without being merged" and leaves `ReviewEnded` unset;
- an approval of the gated head ends Done with the standing read zero times.

Its gate-failure mail assertion takes the new hint and asserts it does not contain "operator".
- `TestFinalize_PullRequest` gains rows over a closed pull request with autonomy true: `ReviewEnded` true halts, and a merge-at-once decision halts, each with no reopen or merge call on the fake API.

**ImpactSummary:** Three branches of the PR-Review row and Finalize, and where the row's wake sends the stays-open mail, change only under a recorded autonomy of true; the hint text change is asserted only by `TestPRReview_Wait`, which this card updates.
