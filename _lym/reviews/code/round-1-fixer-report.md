# Fixer-report, round 1

## Changes

- **F1** (LOW), fixed as proposed, commit `0d030e79`.
  `internal/ideengine/spawn_hub_integration_test.go` gains `checkBeamWithoutReadableOriginOpensBareFolder`, registered as a step in `spawn_scenario_integration_test.go`.
  It marks a pair as a beam, removes its records worktree's lock directory (the `ReadOriginFor` error branch), restores it, then removes the origin record (the `found == false` branch).
  After each row `Spawn` returns no error, `CodeLauncher` has received the pair's bare anchor folder, and no beam workspace file exists.
- Self-check follow-ups, commit `8aabb591`:
  the test file header now names the new check, and the check asserts the lock directory exists before removing it.
- Gate repair 1, commit `80d2d3a6`:
  the check's doc comment now breaks at sentence boundaries, one sentence per line.
- Gate repair 2, one further commit:
  the fabric-vocabulary guard flagged the bare token in the test file, in comments, messages and the hardcoded lock directory name.
  The check now reaches the lock directory through the exported `fabricengine.RecordsLockDirPath`, and its comment and messages say "records lock directory".
  This also removes the hardcoded name the code-correctness lens found unpinned, so the existence assertion now guards the helper's result rather than a literal.

## Self-check

- fix-consistency, LOW, header of the test file omits the new check: fixed.
- fix-code-correctness, LOW, the hardcoded lock directory literal is unpinned: fixed, first by an existence assertion and then by calling `RecordsLockDirPath`.
- fix-code-correctness, note, the absent-record row does not pin the `found &&` guard, since a zero `Origin` has `Beam` false: no change, the row pins the contract and the guard is redundant with the zero value.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- No fork returned without a report.

## Disputed

None.

## Deferred

None.
