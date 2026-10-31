## Changes

- **F1 (MEDIUM), fixed differently.**
  - Added `TestAdoptGateTiers` in `internal/adoptcli/deps_test.go`: rows for an adopt key that reads, an adopt key naming an undeclared tier, a gate config that does not read, and an absent gate config.
  - Proposed row not made: "a gate config without the adopt key returns nil".
    The template supplies a default `adopt` key to every config that loads, so such a config returns the default tiers (`adoptGateTiers() = [0]` in the first run).
    The undeclared-tier row covers the same `TierList` error branch.
  - Reworded the `adoptGateTiers` doc comment, which claimed the no-adopt-key case.
  - The proposed test could not fail on removing the `Tiers` assignment, since `adoptGateTiers` is called directly.
    Extracted the `StartSuite` body into `startCheckWithTiers` and added `TestStartCheckWithTiers`, which fails when the `Tiers` assignment is removed.
- **F2 (LOW), fixed as proposed.** `newCheckFixture`'s clock advances one second per read, and `TestRunCheckMarks` expects tier n's mark to start at read n+1.
- **F3 (LOW), fixed differently.**
  - The command builds its deps from the real hub with no seam, so no tier0 test can run it over fake deps.
  - Extracted the envelope's field building from `cli.go` into `adoptFields`, and added `TestAdoptFields` in the new `internal/adoptcli/cli_test.go`.
    It asserts `message` is present for a started check, `suite_warning` is present for a failed start, and each is absent otherwise.
- **F4 (LOW), fixed as proposed.** `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` asserts the mark's index is below the last `HH:MM` match.
  The first version matched `HH:MM ` with a trailing space; tmux trims the expanded line, so the gate's tier2 run failed on it (`.lym/verify/verify-2.log`).
  The pattern no longer holds the trailing space, and `lym gate test --tier 2 ./internal/reedengine` passes.

Each finding is its own commit. The self-check fixes to F1 are a fifth commit, and the gate repair of F4 a sixth.

## Self-check

- fix-code-correctness, MEDIUM: the `Tiers` wiring is unpinned. Fixed with my own change (`startCheckWithTiers` and `TestStartCheckWithTiers`).
- fix-code-correctness, LOW: the no-adopt-key row was replaced. Already recorded above as fixed differently.
- fix-proposal, MEDIUM: duplicate of the correctness MEDIUM. Fixed with the same own change; the tier1 test it suggested was not made, since tier0 now reaches the wiring.
- fix-proposal, LOW: the `adoptGateTiers` comment named a case that cannot occur. Fixed by rewording it.
- fix-consistency, LOW: the `deps_test.go` file header omitted the tiers test. Fixed.
- fix-consistency, LOW: the `checkFixtureNow` comment no longer matched the advancing clock. Fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
