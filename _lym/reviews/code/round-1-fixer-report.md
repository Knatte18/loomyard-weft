## Changes

All six findings fixed as proposed, except F2, where the proposed option was chosen.
Each fix is its own commit (F3 and F4 share one, since they share a file); the self-check follow-ups are separate commits.
Gate: `lym gate test` over `internal/shedengine`, `internal/shedverbs`, `internal/shuttleengine`, `internal/sheddriver` and `internal/shedarbiter` passed.
A later gate refused two comment lines this round added, one wrapped at a fixed column in `internal/sheddriver/doc.go` and one of 120 characters in `internal/shedverbs/status_test.go`.
Both are now broken at sentence boundaries, one sentence per line, with no change to what they say.
A later verify run failed `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` (tier 2, `internal/loomcli`):
its driver log held an interrupted-step hand-back and not the poisoned status file's "invalid state" text.
This round changed only tests, comments and docs, so no production path of that test moved.
Rerunning `lym gate test --tier 2 ./internal/loomcli` passed the same subtest, so it is a timing flake of the smoke test's driver-log poll and nothing was changed for it.

- **F1** (MEDIUM): `assertCallTimed` in `internal/shedengine/step_test.go` checks `StartedAt` is RFC 3339 UTC and no later than `At`.
  It now covers the done entry, the stuck-bounce entry (`TestStep_StuckWithinBudget`), the stuck-halt entry (`TestStep_StuckWithNoOnStuck`), the reroute entry and the reroute-capped entry (`reroute_test.go`).
  The fold row in `historyfold_test.go` gives each entry a distinct `StartedAt` and asserts the folded entry keeps the first call's.
  Follow-up from the self-check: the no-`OnReroute` wiring-fault entry in `TestStep_RerouteWithoutOnRerouteFailsRun` is covered too.
- **F2** (LOW): fixed with the second proposed option.
  The `IdleGap` comment (`internal/shedverbs/status.go`) and `internal/shedverbs/doc.go` state that calls folded into one stuck entry are one stretch, so the time between them is not reported.
- **F3** (LOW): `TestRun_LoadTurnCappedHold` in `internal/shuttleengine/startup_test.go` has two rows.
  One asserts `Result.APIError` is the newest error text and `NotStarted` is true.
  The other writes the output file during prepare, since the spec refuses a file that exists at start, and asserts the run returns `OutcomeDone`.
  `cappedLoadTurnEnds` scripts the turns for this test and for the capped row of `TestStartup_LoadTurnAPIError`; its last turn carries a distinct error text.
- **F4** (LOW): `TestStartup_LoadTurnAPIError` has a `wantNoticeContains` field; the capped row sets it to the "At N API errors" wording built from `loadHoldMaxAPIErrors`.
- **F5** (LOW): `TestStatusCmd_FoundEnvelope_IdleGaps` in `internal/shedverbs/status_test.go` seeds a ten-minute gap and compares `idle_gaps[0]` to a map of all six JSON keys.
- **F6** (LOW): the capped skill-load hold is named in the `OutcomeAPIError` comment (`internal/shuttleengine/engine.go`) and the api-error description in `docs/overview.md`.
  Follow-up from the self-check: the overview wording ends with "after the hold's back-offs", so it no longer reads as an immediate end.
  The same list sits in `internal/sheddriver/doc.go` and `internal/shedarbiter/doc.go`, whose code branches on `OutcomeAPIError`, so both are updated too.

## Disputed

None.

## Deferred

None.

## Rulings

None. No parent ruling reached this fixer.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
  It noted the no-`OnReroute` entry had no assertion, outside its lens; that is the affected-set LOW below.
- fix-consistency: no findings.
- fix-proposal, LOW: the overview's "so does" inherited "at once" from the model-rejection clause. Fixed as proposed, in its own wording.
- fix-affected-set, LOW: the no-`OnReroute` wiring-fault entry carried `StartedAt` with no assertion. Fixed: `assertCallTimed` added to its test.
- fix-affected-set, LOW: `internal/sheddriver/doc.go` lists the older api-error paths only. Fixed.
- fix-affected-set, LOW: `internal/shedarbiter/doc.go` lists the older api-error paths only. Fixed.
