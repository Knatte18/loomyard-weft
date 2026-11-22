---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The seam is `stopLiveSession func(guid string) error`, which stops only a live strand, wired from `sessionLiveness(c.reed.Status)` and the shuttle runner's `StopStrand`; the review's `stopSession` through the shuttle stop verb left the liveness check unwired, and `pullRequestDeps` fills neither `SessionLive` nor `Sessions` (landingdeps.go:375, sharedbootstrap.go:435). A stop error clears nothing."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The hook is `Admitted func(target string) error`, not `func() error`: `ConsumeTacklerAttempt` and `shedarbiter.ConsumeAttempt` copy a base request built before the target is known and set `Target` afterwards (tacklerconsume.go `request := base; request.Target = envelope.Goto`), so a closure on the base cannot see the target. `shedverbs#Spec` gains `GotoAdmitted`, which `RunGoto` passes, so `specFor` and `driverGoto` reach it; `Spec` joins the Edit targets for that field."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
---

# Round 1 fixer-report

## Changes

- **F1** (card 6): `DriftVerify.passCommand` joins **Edit:** and `storeCaseRulings` joins **Uses:**.
  The ended-ask halt and the `reaskParent` resume now cover both judge paths, `judge` and `passCommand`.
  `TestDriftVerify` gains `DiffCommand` rows for the halt and the re-ask over the same digest.
- **F2** (card 15): `internal/shedrecipe#validPawlDeps` and `#simpleEntryCases` join **Edit:**.
  The Intent says that the fixture stubs the three seams and sets `Paths.DrainMarker`, and that the BeamAwait row's `validatedFields` names the four fields.
- **F3** (card 7): the overview bullet now leaves the `discussion_review_parent_timeout_min` sentence alone.
  It adds a sentence on `pr_review_parent_timeout_min` after the `pr_review_ask_parent` line.
- **F4** (card 6): `reaskParent`'s wait starts at one second and doubles up to a minute while the ask record is unchanged.
  `pattern/PATTERN-daemon-wakeups.md` is under **Extends:** with a bullet for this wait.
  The Intent says that a watchdog kill leaves the ask and the mark open, and that the re-entry continues the wait.
- **F5** (card 11): `PRReview.wake` joins **Edit:**.
  `judge` hands the stays-open mail to `wake`, which sends it after `resumeSession`, or after `judge` when no session was stopped, to the record's `StrandName` at that moment.
  `TestPRReview_Wait` gains a row asserting the resumed session's strand.
- **F6** (card 14), fixed differently: see the frontmatter.
  `clearPRReviewOnGoto` stops a live recorded strand before it clears the record.
  `TestClearPRReviewOnGoto` gains rows for the stop running once before the clear, for a stop error that leaves the record whole, and for a record with no strand.
- **F7** (card 6): `internal/driftengine/doc.go` moved from **Uses:** to **Edit:**.
  The Intent states what the doc says about the forcing, the halt and the re-ask.
- **F8** (card 7): a Bound paragraph is added, stating what the forcing lets through and what bounds it.
- **F9** (card 14), fixed differently: see the frontmatter.
  The card is rewritten around a post-write `Admitted` hook on `GotoRequest` and `RelaunchRequest`.
  `Relaunch` calls the hook only when it moved the run.
  The sites `specFor`, `loomConsumeAttempt` and `goDriverDepsFor` set the hook.
  The guard wrapper and its reset are dropped.
  `TestGoto_Guard` and `TestRelaunch` gain hook rows.
- **F10** (card 14): `Goto`'s comment, and `Relaunch`'s "commits nothing" sentence, name the post-write hook as the one exception.
- **F11** (card 14): the `atOrPastPRReview` seam, its status read and its row are gone.
  `prReviewEntryReset` resets whenever the target is PR-Review.
- **F12** (card 12): the replaced-ask row now says the newer ask ended in a timeout.
  A new row covers a newer ask that is still open and gets the existing "awaits an answer" refusal.
- **F13** (card 5): the two refusals and their rows are dropped.
  A missing number is returned as a caller error with no row.
- **F14** (card 1): `internal/beamsetting/beamsetting.go` joins **Edit:**, and its package doc names `autonomy`.
  The promise to edit `tasksetting_test.go`'s file comment is dropped.
- **F15** (card 10): the prompt-wording assertion is dropped from the autonomy row.
- **F16** (card 13): `internal/loomcli/prcontinue.go` replaces the two member targets.
  The Intent names the header rewrite and the verb's new signature.
- **F17** (card 5): `internal/burlerengine/doc.go` moved to **Edit:**.
  The Intent states that a cleared ended ask answers nothing and that its seat asks again.

## Self-check

The lens reports were saved after I had judged the forks' findings and edited the cards, not before. Each report is still the fork's final message, verbatim.

- fix-consistency, LOW: the twin autonomy-false row of card 10 still asserted the operator block's text. Fixed as proposed: the assertion is dropped.
- fix-consistency, MEDIUM: card 6's open-after-cleared re-entry had no detector and no test row. Fixed as proposed: `reaskWaiting` added, `reaskParent`'s parameter renamed `held`, and a `TestDriftVerify` row added for the state a watchdog kill leaves.
- fix-consistency, LOW: card 6's ImpactSummary was stale. Fixed as proposed.
- fix-consistency, LOW: card 11's ImpactSummary was stale. Fixed as proposed.
- fix-claims, MEDIUM: card 14's stop seam was wired from fields that `pullRequestDeps` does not fill. Fixed as proposed: it is wired from `sessionLiveness(c.reed.Status)` and the shuttle runner, and `sessionLiveness` replaces `pullRequestDeps` under **Uses:**.
- fix-claims, LOW: the doc.go bullet in card 6 left out the no-parent end. Fixed as proposed: it now names timeout, not-sent and no-parent.
- fix-proposal, MEDIUM: the `Admitted` error's way forward did not fit the tackler and arbiter relaunches. Fixed as proposed: the text names `lym loom goto --to PR-Review`, and the card says that those relaunches' refused consume reaches the parent through the stop report.
- fix-proposal, LOW: the re-ask could pair the stored rulings with a retaken snapshot. Fixed as proposed: the re-ask check comes before `resumeJudgeSession` and needs `SnapshotLost` false. With a lost snapshot the judge runs as today, and a row pins this.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
