---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: >-
      The proposed row cannot exist. FirstParentCommits returns only commits newer than a since it has met (internal/fabricengine/parenttips.go:236-247),
      so a root commit never sits in a met range, and a range that does not meet since is refused before the loop.
      The skip of an empty parent in runReachTest was dead, so it is removed instead of pinned.
  - id: F11
    disposition: fixed
---

## Changes

- F1 (`internal/verifytree/reach.go`, `doc.go`): `ReachTurn.findings` writes the union of the listed file's functions and the pending ones, the pending hash winning.
  The `TestWithReachTurn` row `ALaterTurnKeepsTheFunctionsAnEarlierTurnListed` lists A, then B, and a third arrival with A at the same hash gets no turn.
  The `WithReachTurn` comment and the package doc say a turn adds to the listed functions.
- F9 (`internal/verifytree/reach_test.go`): the row `APendingOrTerminalInnerFailureAddsNothingToTheFailingCount` runs a Pending, a Terminal, then a pass at the budget edge, which still gets its turn.
  F1 and F9 share `reach_test.go` and are in one commit.
- F2 (`internal/gatecli/test_integration_test.go`): a `TestGateTest_Scenario` subtest fails a test the prime's shared map holds and asserts the `no change set exists, no parent branch; map commit` line first under it and no `lym gate reach --test` line.
- F3 (`internal/loomcli/wiring_test.go`): `TestWire_ReviewKeysReachTheEnv` asserts `c.env.VerifyReach` and `c.env.WeaverDeps.Geom.ReachPending` are non-nil.
- F4 (`internal/gatefixer/rowgate.go`, `rowgate_test.go`): the live-record path of `spawning` runs the gate through the entry's re-run mark, as proposed.
  A row seeds a live record and asserts the inner call sees the mark on.
  The self-check found the same gap in the entry's final closure, so `waitOnly` takes an `afterFixer` flag that the final closure sets; a row pins it on and off with a live record.
- F5 (`internal/gatecli/reach.go`): a failed `HeadSHA` read lists each package unrefreshed with `reasonHeadUnreadable`.
  The self-check found two sibling branches, the map read and the dirty-file check, that skipped a package with no reason; they list it with `reasonMapCheckFailed`.
  No test forces a HEAD or map read failure in the step, so none was added.
- F6 (`internal/testreach/changedlines.go`, `doc.go`, `export_test.go`, `changedlines_test.go`): `ChangedLines` marks every head line of the stretch between the shared prefix and suffix once its table would pass `maxLineDiffCells`.
  The row `AStretchOverTheTableCapMarksEveryLineOfIt` derives its size from the exported cap and holds one shared line that the diff would leave unmarked.
  The failure message of the table now names the row, since the new row's text is large.
- F7 (`internal/testcover/linemutant_test.go`): the fixture gains a `&&` chain with a constant, a call through an aliased logger import, a local named like the logger package and a rune constant.
  The first row's lines, operators and overlay text cover each, and three rows pin the delete-clause order, the aliased and shadowed logger calls and the rune constant.
- F8 (`internal/gatecli/mutate_integration_test.go`): `TestGateMutate` gains a row for a diff that changes no function, one for a prime whose shared map is stale (`reasonSharedMapStale`), and one for a package with no map (`reasonNoMapToMutate`) that also lands a function whose constant mutant the compiler refuses and asserts it counted unbuildable.
- F10 (`internal/gatecli/reachverb.go`): the dead empty-parent skip is removed, see the disposition's reason.
- F11 (`internal/testreach/index.go`, `internal/gatecli/reachverb_integration_test.go`): `readIndex` reads the file with `os.ReadFile` and `json.Unmarshal`, with no lock, since `WriteIndex` replaces it atomically.
  `TestGateReach` writes an index into the shared directory and removes its lock file, so a locking reader would change the directory the test checks.

Gate repair: four comment lines that were 120 characters or wrapped mid-sentence
(`rowgate.go`, `changedlines.go`, `verifytree/doc.go`, `verifytree/reach.go`) are split at sentence and clause boundaries.
A second gate pass found the `changedlines.go` comment still broke mid-sentence; it is now one sentence per line.

Gates run: `lym gate test` tier 0 over `internal/testreach`, `testcover`, `verifytree`, `gatefixer`, `loomcli` and `gatecli`, and tier 1 over `internal/gatecli`; all passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `internal/verifytree/doc.go` still said the turn records the report's functions; fixed.
- fix-consistency, LOW: `internal/testreach/doc.go` gave the `ChangedLines` summary without the cap; fixed.
- fix-proposal, LOW: F4's missing mark in the final closure; fixed.
- fix-proposal, LOW: F6's cap marks a whole file when a two-site edit spans about 2000 lines; the premise holds, and a per-hunk diff would be a larger change than the finding asked for, so the cap stays and the doc names it (`maxLineDiffCells` line pairs).
  Disposition: fixed, by the documentation.
- fix-affected-set, LOW: F5's sibling branches with no reason; fixed.
- fix-affected-set, LOW: F4's other `waitOnly` callers; the final closure is fixed with the fork's proposal above, and the other entries are left unmarked since they are not the entry whose mark the turn reads.
- fix-affected-set, LOW: F11's `testcover.ReadMap` also reads through a lock; disputed, premise false.
  Every map file is written by `WriteMap`, whose `state.WriteJSON` takes the same lock file first (`internal/testcover/coveragemap.go:182-186`),
  so the lock file exists wherever a map does and a reader creates none.
  The index is the same, and the test removes its lock file after `WriteIndex` only to prove the reader no longer needs it.

## Disputed

Nothing among the review's findings.
The fork finding on `ReadMap` is disputed above, premise false.

## Deferred

Nothing deferred.

## Rulings

No ruling reached me.
