# Fixer-report, code review round 1

## Changes

- F1 (60db0c2b): `TestDriftDeps_AcceptedSeamsKeepAcceptedTestsInTheRun` in `internal/loomcli/driftwiring_integration_test.go`.
  With skip `s1` and accepted test `t1` it asserts that `Compare` skips `s1` and `t1` and renders `{baseline_dir}` as the baseline directory, that `CompareAccepted` skips `s1` alone and renders it as `Paths.AcceptedDir`, and that `CaptureAccepted` skips `s1` alone.
  Commands write their rendered arguments under a temp dir, so no real test suite runs.
- F2 (7766bf19): fixed as the review's fallback.
  `CompareResult` carries an exit code, a log and a reason and cannot name which accepted test differs, so lapse-all stays.
  The Drift-Recheck paragraph of `internal/driftengine/doc.go` now says that under scope full every lapsed acceptance needs the parent's ruling again.
- F3 (4ee36968): a row in `acceptedRows` of `TestRunCapture` runs `sleep 60` under a 300 ms injected timeout, through a new `timeout` field.
  It wants Passed false, a reason naming `timed out after ` and the log.
- F4 (e75b2f4b): `internal/reportengine/doc.go` lists Drift-Recheck in the tapestry webster and darn review phase rows.
- F5 (52031c78): the `enabled` comment of `internal/driftengine/template.yaml` names Drift-Recheck with the other three rows.
- F6 (a17297da): the header comment of `contracts/recipes/tapestry-recipe.yaml` sends a run inside the perch to Drift-Recheck, which sends it back to Drift-Verify while no compare is recorded.
  The `lym loom goto --to Drift-Verify` advice is dropped, since the Recheck's Stuck now does that move.
- Self-check follow-up: the fixture `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` carried the same stale comments, so its main-line list, its Drift-row sentence and its perch-exit sentence now match the recipe.

Gates: `lym gate test` tier 0 over `internal/driftengine`, `internal/loomcli` and `internal/reportengine`, tier 1 over `internal/driftengine` and `internal/loomcli`, and tier 0 over `internal/loomrecipe`, all passed.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the pre-Motif fixture still carried the old perch-exit comment; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, the same fixture's stale "three Drift rows" and perch-exit comments; fixed as proposed, merged with the fix-consistency finding.
