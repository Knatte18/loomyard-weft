# Round 1 fixer-report

## Changes

- **F1** (fixed as proposed): `internal/adoptcli/doc.go` now opens the dobby-start paragraph with "After the resumes".
  Committed with F3 in `56bc060ae`, since both touch `doc.go`.
- **F2** (fixed as proposed): `internal/adoptcli/adopt_test.go` has a new `TestAdopt` row, `PrimeOwnerRecordedAloneStartsTheSuiteLast`.
  Its request sets only `Prime`, and it asserts that `Suite` is non-nil and that `suite` is the last step.
  Commit `9f30db026`.
- **F3** (fixed as proposed): `fabricengine.AdoptScratchDir` sits beside `HubScratchDir` in `internal/fabricengine/junctionnames.go`.
  The `StartSuite` closure in `internal/adoptcli/deps.go` calls it, the unused `lyxdirs` import is gone, and `doc.go` names the accessor.
  Commit `56bc060ae`.

`lyx gate test` on `./internal/adoptcli` and `./internal/fabricengine` is green.

## Self-check

- fix-consistency, LOW: no test pins `AdoptScratchDir`. Fixed with an assertion in `internal/fabricengine/hubscratch_test.go`, beside the `HubLogsDir` and `LogArchiveDir` assertions. Commit follows `56bc060ae`. `internal/fabricengine` is green.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
