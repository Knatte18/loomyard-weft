---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- **F1** (`bafbdfd6a`): added the `BouncerRowGatesAFirstRaisedKeyOnlyWithHardenRound` table to `TestHardenRoundKey` in `internal/shedrecipe/entries_harden_test.go`.
  It builds the Bouncer through `bouncerEntry` with and without `harden_round` and checks round 2's judge prompt for the lighter clause.
  Round 1 is laid out as already judged by `layoutBouncerCirclingRounds`, so one `Call` makes round 2's judge call.
  With the wiring set to false the `harden_round` row failed, and with it restored both rows passed.
- **F2** (`4e15d4e99`): the maintainer comment in `contracts/stencils/bouncer/bouncer-template-judge.md` now says that a Bouncer with `FirstRaisedGates` set renders round 1's rule in every round.
- **F3** (`3baafab0f`): renamed the two rows and the `focusReview` field in `internal/burlercli/validate_integration_test.go`,
  and changed the failure message in `internal/shedrecipe/entries_burlerhalves_test.go`.

Gate runs, all passing: `./internal/shedrecipe`, `./contracts/...` with `./internal/shedadapters`,
and `TestValidateReviewVerb_RoundChecks` of `./internal/burlercli` at tier 1.

## Self-check

No fork findings.
All of `fix-code-correctness`, `fix-consistency`, `fix-proposal` and `fix-affected-set` returned a report.

## Disputed

## Deferred

## Rulings
