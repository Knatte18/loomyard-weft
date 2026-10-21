# Fixer report — webster round 1

## Changes

- F1: `AppendIntegrationTriage` and `triageSectionHead` now say the `final` gate failed. Fixed as proposed.
- F2: removed the `publish_verify` guard row; the `deploy` row pins the same prime-only guard. Fixed as proposed.
- F3: dropped `go_parallel` from the six named fixtures, or replaced it with `tier_parallel` where the value mattered (loomshed round config, configcli machine seed).
  The fix-affected-set fork found five more sites (`gatecli` three, `hubreconcile`, `claudeengine/settings_test.go`), changed the same way in a second commit.
  The `gateslot/config_test.go` rows stay.
- F4: `decodeTier` refuses `covered < 0`; a negative-entry row added to the covers cases.
- F5: the landing tier-timeout halt carries the role clause; the pinning test asserts it.
- F6: `decodeTiers` checks the `Atoi` error and refuses the key; an overflow row added.

## Disputed

- F7, ground `premise false`: the review says `TestAcquire_TakesFreeSlotsUpToLimitThenRefusesAnother` covers the one-slot subtest.
  That test calls `Pool.Acquire`; the subtest is the only one that reaches the `count == 1` branch of `AcquireSlots` (`internal/gateslot/pool.go:165`).
  The subtest was removed, then restored by a revert commit after the fix-proposal fork raised this.

## Deferred

Nothing deferred.

## Self-check

- fix-consistency: no findings.
- fix-code-correctness: no findings.
- fix-proposal: LOW on F7, the single-slot branch is not covered elsewhere; accepted as a dispute, subtest restored.
- fix-affected-set: LOW on F3, five more `go_parallel` fixtures; fixed.
