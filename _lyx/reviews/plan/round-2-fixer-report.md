# Fixer report — plan, round 2

## Changes

- F1, fixed as proposed: card 6's arbiter restart row in `TestGoDriver_Settle` now names its input (an awaiting status at the escalation producer, `Arbiter.EscalationPending` true, `OpenAttempts` returning an arbiter directory and no tackler directory, `AttemptEnvelope` scripted to the saved envelope's path) and asserts the calls are `AttemptEnvelope` and then the arbiter, with no step before them.
  A following sentence says a blocked restart fixture reaches the arbiter only through an applied tackler attempt, so it never reaches `AttemptEnvelope`.
  Premise checked against `decideDrive` in `internal/loomcli/driverules.go` and `resumeOpenAttempt` in `internal/loomcli/drive.go`.
- F2, fixed as proposed: card 6's `HandBackDeps` bullet lists `Park` among the required seams with `OrchRecorded` and `Now`, and "nil parks nothing" is gone.
- F3, fixed as proposed: card 13 names the sources of `ReadStatus`, `RunLockHeld`, `Pause` and `PauseHold`, and says battencli has no pause-hold today.
  `TestBattenDriverDeps` gains an assertion that every required `Deps` seam card 6 lists is non-nil.
- F4, fixed as proposed: card 4's three per-recipe `TestTacklerPrompt` rows are replaced by one row that renders a told recipe line under "The run", and the card says card 8's `TestTacklerFor` and card 13's `TestBattenDriverDeps` pin the per-recipe lines.
- F5, fixed as proposed: card 6's `ArbiterOutcome` gains `Verdict` and `Role` strings copied by loom's wiring, and the moved `arbitrate` fills the attempt note from them and tests `Resume`, so `internal/sheddriver` names no `loomengine` arbiter word.
- F6, fixed as proposed: card 13's `driverDepsFor` Create target drops the `release` parameter, and `driveRun` ignores the `release` that `sheddriver.DriveVerb` hands its callback.

`lyx loom validate-plan` over the edited plan returns `ok: true` with informational findings only.

## Disputed

None.

## Deferred

None.
