# Pin pr-findings' landing wiring on a darn run

Issue #187 reported `lym loom pr-findings` failing on a run's PR-Review row with `NewParentAlign: Deps.OpenFabric must not be nil`.
That failure is the one 6fa7f1740 (closes #155) fixed:
the verb builds the run's shed to read the PR-Review row's send-back edge,
and since that commit `(*loomCLI).prReviewSendBackEdge` wires the landing seams before it builds the shed.
The run that filed #187 used a lym binary that can no longer be traced, and the installed lym already contains 6fa7f1740,
so the report came from a binary built before the fix.

The fix was pinned for a tapestry run only, whose first landing row is Parent-Align.
This change adds the darn recipe, whose first landing row is Finalize, so both recipes' landing wiring is pinned.
The entry is closed as overtaken by 6fa7f1740, and no production code changes.

## Changes

- `internal/loomcli/prfindings_integration_test.go`: `TestPRFindingsCLI_SendsBackOnATapestryRun` becomes `TestPRFindingsCLI_SendsBackBlockingFindings`, a table over the tapestry and darn recipes.
  Each row seeds the template pair's run with its recipe, feeds `pr-findings` a BLOCKING findings file through the real cobra tree, and asserts `send_back: true`.
  The hub setup moves into the helper `blockingFindingsRun`.

## Design decisions

- Following the parent's recorded decision, the darn row is a table row of the existing test, not a second test:
  both rows pin one behavior, that the verb builds the shed with its landing rows wired.
- The task is labelled `bug`, but the new row passes on the merge base, because 6fa7f1740 already fixed the defect.
  The parent recorded this as a no-test-seam decision: the entry lands as overtaken, with the test as a guard.

## To check

- That both rows pass at tier 1: `lym gate test ./internal/loomcli --tier 1`.

## Merge danger

**Door:** two-way
**Blast radius:** tests
