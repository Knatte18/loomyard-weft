# Settings to send code review and drift judge rulings to the run's parent

Before this change, a run's parent could be asked by setting only at PR-Review (`pr_review_ask_parent`) and Discussion-Review (`discussion_review_ask_parent`).
Two new settings now route the Code-Review perch's and the drift judge's rulings to the parent orch.
Both are off by default and settable per task in `run_settings`.

## Code-Review: `code_review_ask_parent`

The review row's parent seat (`shedadapters.ParentReviewSeat`) used to be Discussion-Review only.
It now takes a line tag, a subject and an `AskEachRound` switch, and `loomcli.parentSeats` builds a seat for each of the two segments.

- With `code_review_ask_parent` on in `loom.yaml`, every Code-Review round mails the parent orch a `[code-review request]` line, signed as `<shortname>:LOOM`.
  The line names the task worktree, and from round 2 on also the previous round's review and fixer report.
  It also names the deadline and `lym loom parent-review answer <slug> --ask <n>`.
- As with Discussion-Review, the round's chair waits with `lym loom parent-review await` and keeps every parent finding as `origin: parent`.
  The parent-coverage gate holds the review to each report id, the fixer fixes the findings, and the Bouncer judges the round.
- `code_review_parent_timeout_min` (default 30) sets the deadline.
  `loomengine.CapParentReviewWait` now takes the perch and caps this wait against `review_timeout_min` the same way as the discussion key.
- The Discussion-Review tag is renamed to `DiscussionParentAskLineTag`, and its line text is byte-for-byte unchanged.

## Drift judge: `judge_ask_parent`

With `drift.yaml`'s `judge_ask_parent` on, Drift-Verify (and Drift-Recheck) adds a `parent-agreement` gate entry to the judge's table, after the `judgement` entry (`driftengine.ParentAgreementGateEntry`).

- The first time a judge spawn's turn ends with a judgement.yaml that parses, the entry records an ask in the same never-tracked ask record the review rows use.
  The ask's seed names the spawn id.
  The entry mails the parent one `[drift-judge request]` line naming judgement.yaml and a summary of the rulings.
  The summary counts cells and files by ruling, and names each cell that does not match, each case and each file with an uncaused check.
- The entry fails, telling the judge to wait with `lym loom parent-review await`, until the ask ends.
  A report with findings (rulings the parent wants changed) holds the judge until judgement.yaml is rewritten after the answer.
  A satisfied report, a timeout, or an ask that never reached the parent passes.
- `judge_parent_timeout_min` (default 30) sets the deadline.
  Once a spawn holds an ask, the entry stays on whatever the key now reads.

`contracts/orchrole/orch-role.md` gains a section for each new request, and `lym loom parent-review`'s help no longer says Discussion-Review only.

## Design decisions

- **The code-review ask goes out at round start, beside the reviewer, not between the fixer and the Bouncer.**
  The issue asked for the ask to name the round's review and fixer report and to wait before the Bouncer judges.
  This change reuses the Discussion-Review machinery instead: the ask record, the answer/await verbs, chair consolidation, the coverage gate and the capped deadline.
  To keep the parent seeing each ruling, every round's ask names the previous round's review and fixer report.
  The trade-off: the last round's fixer report goes out with no later ask, so the parent never sees it before the Bouncer approves.
  A wait before the Bouncer would need the Bouncer to take in parent findings that arrive after the fixer, which no current path supports.
- **The drift keys are `judge_ask_parent` and `judge_parent_timeout_min` in `drift.yaml`, not `drift_judge_ask_parent`.**
  This follows the module's naming of `judge_model` and `judge_interactive`.
  Per task they are set as `{"drift": {"judge_ask_parent": true}}`.
- **The drift ask names a summary, not every cell.**
  Every ruling and its reason is in judgement.yaml, whose path the line names.
- **Bounds.**
  The drift entry checks that the judge rewrote judgement.yaml after a report with findings, not that it applied each one.
  The drift deadline is not capped by the judge's own timeout.
  A judge that times out while waiting is resumed under the same spawn, which still holds its ask.

## Tests

- `internal/shedadapters`: a new test shows that a seat asking each round asks again after a satisfied round, with the line naming the previous round's review and fixer report.
  The existing asking-rule and line tests now use the new seat fields.
- `internal/loomcli`: placement covers both seats on their own segments, and `askParentSetting` reads the key it is given.
- `internal/loomengine`: new rows cover the code keys' defaults, their override and the timeout refusal.
  `CapParentReviewWait` gets code-perch rows.
- `internal/driftengine`: `TestParentAgreementGate` covers no ask, open, past deadline, satisfied, findings before and after the rewrite, no parent and a failed post.
  `TestJudgeAskLine` and `TestJudgementSummary` pin the line and the summary.
  New config rows cover the judge keys.

I compiled and vetted these packages, but I did not run the tests in this session: the verify gate runs them on the committed tree.
