---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

## Changes

- F15: `darnTable` in `internal/loomcli/darnwiring.go` loads the gate config and calls `MeasureCall` only when the writer is gate-fixed.
  `DarnTable` requires `MeasureCall` only when `GateFixed` is set; the field comment and package doc say so.
  Tests: the empty-`MeasureCall` refusal row sets `GateFixed`; a new ungated `TestDarnSpec` row builds with it empty;
  the wiring measuring-call subtest gains an ungated no-`{args}` row that builds, a gated no-`{args}` row that fails, and an absent-`gate.yaml` gated row that fails.
  The `wiring_test.go` row that expected an ungated table to fail on an absent `gate.yaml` now expects it to build, and its dead `wantTableErr` field is gone.
- F16: deleted `TestVSTestCounterReplacesAnInvocationsConsoleCounts` and `TestRemoveResultsDir` and the unused `io` import.
- F17: `Config.MeasureCall` runs its `{args}` and target checks over `tiersRunBy`, which sorts, compacts and drops covered tiers as `dropCovered` does.
  The doc comment and `gateslot/doc.go` say a covered tier counts for neither.
  Test rows `covered` and `coverednoargs` added to `TestConfig_TierUse`.
  The review's example row used tiers 0 and 1; the fixture's `tier1` and `tier3` are taken, so the rows use a new `tier4` covering 0 and 2.
- F18: the expected words in `TestRunTierReadsTheVSTestResultsFile` are literals.
  That test is tier1 and the gate run covered tier0 only; the literals equal what `vstestResultsArgs` returns.

Gate comment lint: the added line in `internal/gateslot/doc.go` reached 120 characters; it is now broken after its first clause.

Tier0 gate runs over `gateslot`, `darnengine`, `loomcli` and `gatecli` pass.

## Self-check

- fix-consistency, LOW, `darnengine/doc.go` package doc still says every caller tells the measuring call: fixed.
- fix-consistency, LOW, same paragraph's required-field sentence: fixed with the same edit.
- fix-code-correctness, LOW, no test pins an absent `gate.yaml` failing a gated table: fixed with a new wiring row.
- fix-proposal, MEDIUM, F16 deletion leaves `replaceInvocation`, `startInvocation`, `resultsFileLines` and `removeResultsDir` with no tier0 test, so the gate's reach report lists them again: deferred.
  The reviewer ruled the tests duplicates under PATTERN-test-speed, and keeping a tier0 test would reinstate that duplicate.
  Whether the reach report or the duplicate rule wins is a tradeoff for the parent.
- fix-proposal, LOW, `tiersRunBy` duplicates the `dropCovered` rule: deferred.
  `gatecli` imports `gateslot`, so sharing one implementation means moving `dropCovered`'s selection into `gateslot` and rewiring `gatecli`, which is a refactor the review did not ask for.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The two fork findings above marked deferred, with the reasons given there.

## Rulings

None.
