# Pin pr-findings on a darn run with a CLI test

`lym loom pr-findings` was reported to fail with `NewParentAlign: Deps.OpenFabric must not be nil`.
The base already fixes this: commit 6fa7f1740 wires the landing seams before `pr-findings` builds the run's shed.
The failing binary cannot be traced, and only a tapestry run had a test pinning the wiring.
This change adds the missing darn-run test and changes no production code.

Closes #187

## What changes

`internal/loomcli/prfindings_integration_test.go`:

- `TestPRFindingsCLI_SendsBackOnATapestryRun` becomes `TestPRFindingsCLI_SendsBackBlockingFindings`, a table test with a `Tapestry` and a `Darn` subtest.
- Each subtest seeds a run of its recipe, then drives `pr-findings` through the real cobra tree with a BLOCKING findings file.
  It asserts exit code 0 and `send_back: true` in the envelope.
- The recipes differ in their first landing row: Parent-Align on tapestry, Finalize on darn.
  Each refuses to build without the landing seams, so the two subtests cover both wirings.
- The shared setup moves to a helper, `blockingFindingsRun`, which returns the code worktree and the findings path.

## Design decisions

- Both subtests pass on the base, so the entry lands as overtaken by 6fa7f1740, with the test as a guard.
  No test can fail on a base that already holds the fix, so there is no failing-first seam.
- The subtests run in parallel and share no state: each copies its own hub.

## To check by hand

- Finding `findings-path-built-twice` (consistency, `internal/loomcli/prfindings_integration_test.go`) is fixed in this diff:
  `blockingFindingsRun` builds the findings path once and the subtest no longer rebuilds it.
  Confirm no second construction of that path remains.
- Nothing was run beyond the tier-1 gate on this test.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
