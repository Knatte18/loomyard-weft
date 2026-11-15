# Fixer report — plan, round 2

## Changes

- **F1** (card 11), fixed as proposed.
  `planPass` keeps `run.CancelMarker` in its first gate beside `drivable`, and the act rule takes the place of its `Held` check alone; the Intent says why: `runfsm.Judge` reads the batten record alone and answers act between the cancel lock's take and the intent's write, which only `CancelMarker` sees (card 8).
  `drivePasses` drops the pending lift on `!drivable(run) || run.CancelMarker || verdict is not act`.
  `planPass`'s doc names the act verdict where it names the adopt pause and the cancel marker as a cancel under way.
  `TestPlanPass` gains a row where `CancelMarker` is true with an act verdict, giving none on every trigger.
  The `RunView.CancelMarker` doc bullet is unchanged.
- **F2** (card 5), fixed as proposed.
  The `PastParentMerge` input names the `restepped` source (`sheddriver.ReadSettleRecords` over `shedrun.SettleRecords(task, shedrun.SelfRunID)` then `sheddriver.RestepRecorded`, as `childRestepped` does), and states that with no pair here and the child read through `fabricengine.OriginRunStatus` it is false with neither seam called.
  The fixer worded the reason beyond the reviewer's text: an owned-elsewhere run is handed over before the input matters, and for any other run false only withholds the act a hold would otherwise yield.
  `TestReadRun` gains the row of a running child at Finalize read from origin with no pair here: `PastParentMerge` false, no error, neither seam called.
- **F3** (card 5), fixed as proposed.
  `TestReadRun` sits in the internal package `pairteardown`; `TestReadRun_ReadsTheSameStateWithoutTheCache` sits in `pairteardown_test`, as `teardown_integration_test.go` does, and calls the exported `pairteardown.ReadRun`.
- **F4** (card 5), fixed as proposed.
  The `**Edit:**` line `internal/pairteardown/followup.go` became `internal/pairteardown#landRow` and `internal/pairteardown#worktreeTeardownRow`.
  `lym loom validate-plan` reports `ok: true` with informational findings only.
- **F5** (card 3), fixed differently.
  The terminal `Landed` clause now names a way forward that lifts the cause.
  The proposed wording was not carried over whole, for two reasons the self-check found:
  - The proposal said it matched the done-slug refusal of `lym batten run`, but `doneSlugRefusal` (`internal/battencli/arm.go:848-853`) names `lym batten cleanup --apply` and no pair removal or board release.
    Without the cleanup step, a `Landed` record that stands (`Derive` rule 1) reads `Landed` again on the re-run.
  - The proposal had the board release apply to every landed slug, but `setStatusReleasingProject` (`internal/boardengine/projectrelease.go:16`) refuses a null status on a `done` project, so a landed beam could not follow it.

  The clause now names `lym batten cleanup --apply`, then `lym fabric remove <slug>` if the pair still stands, then the release for a task only, then deleting the leftover branch locally and on the remote.
  The alternative is giving the new task another slug, which is the one way forward for a landed beam.
  The `TestAct` line says the terminal `Landed` row's text names each of these steps.

## Self-check

- fix-consistency, MEDIUM: the `Landed` clause's "as the done-slug refusal words it" was false against `doneSlugRefusal` and card 10. Fixed with own change (the F5 rewording above; the cross-reference dropped, the cleanup step added).
- fix-consistency, LOW: card 11's `doc.go` pass-planning bullet credited the act rule alone. Fixed as proposed: it now says the act rule decides beside a cancel under way, which gives no pass.
- fix-consistency, LOW: card 11's `strandedEnded`/`readWaits`/`runNotices` comment bullet and `doc.go`'s other paragraphs named the unfinished cancel intent alone. Fixed as proposed: they name a cancel under way, an unfinished cancel intent or a cancel lock another process holds.
- fix-claims, LOW: the same false cross-reference, plus no run-directory removal for a `Landed` record that stands. Duplicate of the fix-consistency MEDIUM; fixed by the same change.
- fix-proposal, MEDIUM: the board release refused for a landed beam. Fixed with own change: the release applies to a task only, and another slug is named as the beam's way forward.
- fix-proposal, MEDIUM: following the wording with a batten record standing leads into `doneSlugRefusal`. Duplicate of the fix-consistency MEDIUM; fixed by adding `lym batten cleanup --apply`.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
