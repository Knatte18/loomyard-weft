# umbrella-anchor: every batch recorded done

Batches 01 and 03 were already done when this session started.
Batch 06 was reset to its start by the parent and re-run in this session; the other five were run here.
No digest reported a `Pattern-Departure:` trailer or a rename.

## Batches

- 06 `clone-records-marker-on-prime-weft`: done, head `f18068daa`.
  Its fork wrote cards 6 and 7, as the batch prompt lists both.
  Deviations: `internal/fabriccli/cleanhub_integration_test.go`, `internal/fabricengine/add.go`, `clone.go`, `clone_adopt_test.go`, `clone_emptyweft_integration_test.go`, `livestate_verbs_test.go`, `recordsprobe.go`, `remove_junctions_integration_test.go`.
  `record-batch` warned that two fork transcripts counted toward the batch: a stray fork I spawned before `begin-batch` and stopped with the worktree still clean, then the real one.
- 08 `reconcile-migrates-markers`: done, head `e20a42d65`.
  Deviations: eleven files in `internal/fabricengine` and `internal/fabriccli`, mainly the new `anchorstop.go` and its tests.
- 11 `origin-reconcile-stops-on-marker-change`: done, head `793a2ea9c`.
  Deviations: `internal/battencli/cancel.go`, `internal/beamcli/promote_test.go`, and four files in `internal/fabricengine`.
  Its batch gate failed on one test, see the verify gate findings below.
- 13 `move-record-stops-weft-writers`: done, head `e8e6cc268`.
  Deviations: `internal/gitexec/gitexec.go` and seven files in `internal/fabricengine`.
- 15 `anchor-change-engine`: done, head `7a3fa1748`.
  The first fork reached its turn limit without a report and left cards 15 and 16 committed; one re-fork finished card 17.
  Deviations: fifteen files across `internal/dobbyengine`, `internal/fabriccli`, `internal/fabricengine` and `internal/hubgeom`.
- 18 `beam-create-at-an-anchor`: done, head `b4a7a5402`.
  Deviations: `internal/adoptcli/adopt_test.go`, `internal/adoptcli/refusal.go`, `internal/beamcli/create_test.go`, and three files in `internal/fabricengine`.

## Verify gate findings

The verify gate failed on attempt 1 of 2 with one failing test: `internal/beamcli.TestDobbyRuns_PairAtItsOwnAnchor/InvalidMarkerRefusesTheRunsAndLeavesThePairOutOfTheWatch`.
The findings file named cards 01 and 11 as the cards whose commits touched the failing package.
One fixer fork was spawned; the gate verifies again after this turn ends.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `cf28d72035bcb2f7a74c9a260acfb8720ac6e32b` fix: dobby run listing refuses a pair with an invalid anchor marker instead of leaving its run out
