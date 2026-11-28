---
dispositions:
  - id: F8
    disposition: fixed
---

# Round 2 fixer report

## Changes

- F8 (commit `14d45ead6`): `TestRunAutonomyStamps_PostsARefusalOncePerEpisode` in `internal/beamcli/dobbyautonomystamps_test.go` now opens with `tiergate.Require(t, 0)`, and the file imports `tiergate`.
  `lym gate test ./cmd/lym ./internal/beamcli --tier 0`, filtered to `TestTierTags_OneTierPerTestFile` and the changed test, ran both and passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
