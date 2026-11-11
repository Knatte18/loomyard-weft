# Fixer report, round 1

## Changes

- F1 (MEDIUM, fixed as proposed): added `TestHubLandDeps_HeldSlugs` to `internal/beamcli/deps_integration_test.go`.
  It builds a real hub, creates the run directory `held-run` under the prime's shed directory and the task branch `held-branch`,
  and asserts `hubLandDeps(prime).HeldSlugs()` holds both.
  Commit `ba000bb5`.
  The tier-1 gate over `./internal/beamcli` passed, and the new test ran and passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
  Its remarks that a prefix such as `held-` and a swallowed error would pass the test are outside the review's proposal, and no finding.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
