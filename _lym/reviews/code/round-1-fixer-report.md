# Fixer-report, round 1

Gate: `lym gate test` tier 0 and tier 1 over `adoptcli`, `pauserecord`, `beamcli` and `tendercli` pass.

## Changes

- F1, fixed differently: `runWorker` clears the recorded steps only when the call restarts the adopt.
  Not proposed: the reviewer's change alone leaves the pause lost, since a re-run `pause` step replaces the earlier part and skips listed slugs.
  So `stepPause` now reports `AdoptPaused`, the paused run state and `PauseStanding` for every slug in `Inputs.Paused`.
  Tests: a resume restarting at reconcile, and one stopped at reconcile, both keep the earlier pause in the envelope.
  The pairs mutations are re-produced by the re-run `pairs` step, so no row pins them separately.
- F2, fixed as proposed: `stepDone` renders keep and drop from `run.Req` with only the local-commit flag changed.
  A test feeds the stop's own keep-local command through `judgeStanding`.
  The `adopt_test.go` row for two slugs expects the new text.
- F3, fixed as proposed: `StoppedStepPartLeavesTheRecordOnceTheStepSucceeds` in `worker_test.go`.
- F4, fixed as proposed: a resume takes slugs and root from `record.Subjects` (`splitSubjects`).
  Own addition from the self-check: a recorded beam the board no longer holds is dropped.
  Test: `ResumeOfAnAllAdoptKeepsTheRecordedSubjectsWhenABeamIsMadeMeanwhile` covers a beam added and one removed.
- F5, fixed as proposed (state it): `pauserecord/doc.go` says the parent-lock exemption and release deliberately ignore a converted adopt entry.
- F6, fixed differently: not proposed, a row in `dobbyengine/owned_test.go`, since that test drives a fake lift seam.
  The entry filter lives in `liftOwnerPause` in `beamcli`, so `internal/beamcli/dobbydeps_test.go` pins that an adopt and a hold entry stay standing.
- F7, fixed as proposed: `TestPause_AnAdoptPausedSlugAnswersAlreadyPausedAndKeepsItsEntry` in `tendercli/cli_test.go`, as its own test over a fresh hub.
- F8, fixed as proposed (state it): `doc.go` and the `reconcileWire` comment say `reconcile.mutations` is empty and the top-level field holds the record.
  The done-envelope test asserts it over a reconcile report that carries a mutation.
- F9, F10, fixed as proposed: `//testtiming:keep` lines.
- Self-check follow-on: the per-slug way forward in `takeOverSlug` now names the adopt's own request, as `stepDone` does, since the project-only command is refused by `judgeStanding`.

## Self-check

- fix-consistency, LOW, restated pause undocumented: fixed, `adoptcli/doc.go` step pause.
- fix-consistency, LOW, per-slug way forward names another command than the stop: fixed, `adopt.go`, `adopt_test.go`.
- fix-consistency, LOW, reader-clarity note on kept steps: no change, it names no contradiction.
- fix-affected-set, LOW, restated pause standing for a lifted slug: no change, `stepBeam` and `resumeSlug` treat `Inputs.Paused` alike; documented.
- fix-code-correctness, LOW, F1 row passes with the defect: fixed, added the stopped-at-reconcile row that fails with Steps cleared.
- fix-code-correctness, LOW, flags apply to the whole adopt: no change, needed for `SameSubjects`; a subject already owned here skips the drop.
- fix-proposal, MEDIUM, resume of a removed beam: fixed, gone slugs are dropped and the F4 row covers it.
- fix-proposal, LOW, drop-local over the whole adopt: same as above, no change.
- fix-proposal, LOW, `doc.go` line on the reconcile error's mutation record: no change, the top-level mutation record still holds it.
- fix-proposal, LOW, F5 doc lacks the lock staying held: fixed in `pauserecord/doc.go`.

- Gate repair: three comment lines the gate flagged (`adoptcli/doc.go`, `adoptcli/steps.go`, `pauserecord/doc.go`) were split into one sentence per line, under 120 characters.
  No wording changed beyond the split.

## Disputed

None.

## Deferred

None.

## Rulings

None.
