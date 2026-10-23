# Plan round 1 — fixer report

Every finding's premise held against the plan and the code; all thirteen are fixed.

## Changes

- **F1** (card 26), fixed as proposed: right after `readBefore`, `Call` reads the entry's attempt record and, when the merge record holds no aligned tip and the attempt record no `AlignFrom`, writes the task's fork point, `MergeBase(ParentBranch)` before the merge, as `AlignFrom`.
  `TestMergeIn_Fixer` gains the row where a first Merge-In stops with a `landing` gate infrastructure error after the merge, and the resumed entry, whose merge is up to date, still spawns the `merge-in` fixer.
  Self-check wording: the fork point is named as `MergeBase(ParentBranch)`, not as what `readBefore` read, since card 24 narrows `readBefore`.
- **F2** (cards 10, 11, 26, 29), fixed as proposed, then extended by the self-check: card 10 states that an entry has no attempts left when `Spent` reaches its row's cap, whatever `Halt` notes.
  Card 11 states that `run` is never called with zero turns left.
  Card 26's resume with no attempts left keys on `Spent`, and card 29 gains a defined resume: over a failing gate it reroutes or halts as a spent cap does.
  After the self-check, over a passing gate both cards judge `Baseline..HEAD` by the caps before Done or the landing.
  Merge-In judges under `CodeReviewOff`; Finalize reroutes or halts.
  Both tests gain a row for a resume after a last-turn timeout.
- **F3** (cards 10, 26, 29, 34), fixed as proposed: the undecodable-attempt-record halt opens with `LandingStopPrefix`, so the hand-back mark sends it to the orch.
  `TestHandsBack` gains that reason at Merge-In and at Finalize.
- **F4** (card 22), fixed as proposed: the Code-Review rubric's measuring-stick section, Do-not-flag rule and Also-flag section carry D5.
  The measuring stick is the decision record and the plan, and the review covers the code in general as well as against both.
- **F5** (card 51), fixed differently after the self-check.
  `TestResumeVerb` and `TestLoopLockExit` join the Edit list.
  `TestResumeVerb`'s own-leftover row names Code-Bouncer, not Finalize, because Finalize merges the parent in under this plan (cards 27 and 28).
  `TestLoopLockExit` drops its two PR-Gate awaiting rows instead of retargeting them: after card 41 they would pin the release that its `awaiting at Describe` row (`looplock_test.go:74`) already pins.
- **F6** (card 13), fixed as proposed: `TestWire_DarnRecipe` joins the Edit list, and its `darnSeams` list shrinks to `DarnTable`, `Darn.Commit` and `Darn.LatestOutcome`.
- **F7** (card 26), fixed differently: a nil `CodeReviewOff` reads as budget 0, so the caps apply.
  A failed config read is an error stop naming the config, not budget 0 as proposed: an unreadable `loom.yaml` is an operational error, and a budget-0 reading would surface it as a misleading over-cap halt.
  `TestMergeIn_Fixer` gains a nil-seam row.
- **F8** (card 21), fixed as proposed: later D14 retirements go to card 23's drop list, which runs after the rename table.
- **F9** (card 15), fixed as proposed: `TestVerifyGate_RoundScenario` drops its failure-record rows and `writeFailure` helper, so card 46's rename finds no `PublishFailure` reference there.
- **F10** (card 47), fixed differently in placement only: `internal/landingshed/doc.go` goes under **Extends**, not **Edit**, as every other card lists a `doc.go`.
  The `require_pr_to_base` pull-request paragraph is replaced by one saying `deploy_branches` is read only by batten's Land row.
  After the self-check, the hub-wide landing-config paragraph (`doc.go:108`) names `deploy_branches` too.
- **F11** (card 52), fixed as proposed: `shed-recipe-spec.md` names `Env.ReviewInput` where it named `Env.RejectReview`.
- **F12** (card 11), fixed as proposed: the closure measures a turn's duration through the `verifyGate`'s injectable `now` clock (`verifygate.go:157`), and `fakeFixerSeats` advances that clock by the scripted turn duration.
- **F13** (card 29), fixed as proposed: the verified-tree row leaves `TestFinalize_GateAndFixer`, since `TestFinalizeVerify_Lands`' `verified tree skips` row (`finalize_verify_test.go:98`) pins it.

Further self-check changes: card 52's `docs/overview.md` batten `Land` sentence deploys on `deploy_branches`.
Card 52's orch-rule list of landing-stop causes gains the undecodable attempt record and its way forward.

## Self-check

- fix-consistency, MEDIUM: an exhausted Finalize resume over a passing gate landed `Baseline..HEAD` unjudged by the caps. Fixed with own change (merged with fix-proposal 1): both cards judge the change by the caps on that path.
- fix-consistency, LOW: card 52's orch rule omitted the undecodable-attempt-record halt from the landing-stop causes. Fixed as proposed.
- fix-claims, LOW: card 26 named the fork point as what `readBefore` read, which card 24 narrows. Fixed with own change: the card names `MergeBase(ParentBranch)` before the merge.
- fix-claims, LOW: `TestLoopLockExit`'s retargeted rows would duplicate the Describe row. Merged with fix-proposal 3.
- fix-proposal, MEDIUM: the exhausted-entry path over a passing gate skipped the caps at Merge-In and at Finalize. Fixed as proposed.
- fix-proposal, LOW: a failed `CodeReviewOff` config read read as budget 0. Fixed as proposed: it is an error stop naming the config.
- fix-proposal, LOW: `TestLoopLockExit`'s retargeted awaiting rows duplicated an existing row. Fixed as proposed: the two rows are dropped.
- fix-proposal, LOW: Finalize is a poor "another row" for `TestResumeVerb`'s own-leftover row. Fixed as proposed: it names Code-Bouncer.
- fix-affected-set, LOW: `internal/landingshed/doc.go:108` keeps `require_pr_to_base` in the hub-wide landing-config paragraph. Fixed as proposed in card 47.
- fix-affected-set, LOW: `docs/overview.md:641` keeps `require_pr_to_base` in batten's `Land` sentence. Fixed as proposed in card 52.

## Disputed

None.

## Deferred

None.
