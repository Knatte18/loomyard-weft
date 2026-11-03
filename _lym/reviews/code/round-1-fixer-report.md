## Fixer-report, round 1

### Changes

- F1: added a `TestDriftBaseline` row, "an existing baseline.md is kept under a configured scope, with no agent". It seeds a scope record and a `baseline.md`, sets `configScope`, and asserts the file is untouched and no agent ran. Commit `312f17ef`. Fixed as proposed.
- F2: `configuredBaselineReport` no longer claims where the scope came from; it says the report was generated, no baseline agent ran and no per-test drift is expected. The configured-full row now asserts "no baseline agent ran". Commit `312f17ef`. Fixed as proposed.
- F3: the Drift-Verify stencil line for `baseline.md` gained a clause saying no per-test drift is expected from a report that names no agent. Commit `67cc6ea7`. Fixed as proposed; the self-check then reworded the clause (below).

F1 and F2 share `baseline_test.go` and went in one commit.

### Self-check

- fix-proposal, LOW: the doc comment of `configuredBaselineReport` still named the drift config as the scope's source. Fixed, with the self-check commit.
- fix-proposal, LOW: the F3 clause keyed on "the scope was configured", which is false when a parent or agent wrote `baseline.md`. Fixed by keying it on the report's own text ("when the report says no baseline agent ran"), as the fork proposed.
- fix-affected-set, LOW: the `BaselineReport` field comment in `paths.go` called it the agent's report only. Fixed.
- fix-code-correctness: no findings.
- fix-consistency: no findings.

All forks returned a report; the four are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

### Disputed

None.

### Deferred

None.

`lym gate test ./internal/driftengine ./contracts/stencils` passes at tier 0.
