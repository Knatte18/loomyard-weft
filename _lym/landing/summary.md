# Deliver the PR-Review re-ask and record a refused post as undelivered

Fixes [issue #200](https://github.com/Knatte18/loomyard/issues/200).

Under `pr_review_ask_parent`, the PR-Review session asks its parent orch again after each set of fixes with `lym loom parent-review ask`.
That verb runs in the session's own pane, and it signed the post as the hub's `LOOM` program.
Creel lets only an orch strand's pane claim a program name, so every re-ask was refused.
The asker then ended the refused ask `no-parent`.
The session read that as a parent that is down and ended its turn, so the review stalled while the parent was live.

## What changes

- **The session's re-ask is delivered.**
  `prReviewAsker` takes a strand claim.
  `parent-review ask` passes the session's own strand, and the post goes out through the new `orchcli.PostAsCaller`, signed as that strand.
  `PostAsCaller` uses the same strand-first signing as `orchcli`'s notices, falling back to the program when creel places the caller outside the strand's pane.
  That signing now lives in `postSigned`, shared with `notifyPrime`.
  The PR-Review row's own first ask passes no claim and still signs as `LOOM`.
- **A refused post is a delivery failure, not `no-parent`.**
  `burlerengine` gains the end state `undelivered`: a resolved parent orch whose post creel refused or that failed, with the error as its reason.
  The PR-Review asker, the review rounds' parent seat (`shedadapters`) and the drift judge's seat all end a failed post `undelivered`.
  `no-parent` keeps its meaning: no parent orch resolved, and nothing was posted.
- **Readers of the state.**
  `parent-review await` answers an undelivered ask's reason, as it does a not-sent one's.
  Drift-Verify treats an undelivered judge ask as it treats a `no-parent` one: under autonomy it halts and the resume asks again.
  The autonomous PR-Review block tells the session that an undelivered end is a refusal, not a down parent:
  it sends the reason and ask number to its parent with `lym creel post --parent`, then waits for mail.
- **Orch role.**
  `orch-role.md` says a `[pr-review request]` comes from `<shortname>:LOOM` or from the run's PR-Review session asking again.

## Design decisions

- Signing as the session's own strand is used rather than sending the re-ask through the row.
  The verb already checks that its caller is the review's PR-Review session, and a strand-signed post needs no new channel between the session and the driver.
- An older binary that reads an `undelivered` ask counts it as ended, since `ParentAskState.Ended` treats any state it does not know as ended.

## Tests

- `TestPRReviewAsker_Ask`, `TestParentAgreementGate` and `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut` have a failed-post row.
  Each row now expects `undelivered` with the delivery error as reason.
  Each fails on the merge base, which recorded `no-parent`.
- `TestDriftVerify`'s autonomy rows gain an `Undelivered` row: a judge ask whose post fails halts under autonomy and stores the rulings otherwise.
- `TestParentReviewAwaitVerb` gains a row answering an undelivered ask's reason.
- `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller` drives `prReviewAsker`'s ask through creel's daemon run in process.
  With the test process listed as the session's pane, the ask is delivered signed as the session's strand.
  With no claim, it is signed as `LOOM`.
  Either way the ask is recorded open with the configured deadline.
  The test's daemon helper, `startNoticeOrch`, now lists extra strands and returns its delivery and the orch's name.

## Merge danger

**Door:** two-way
**Blast radius:** parent-review
