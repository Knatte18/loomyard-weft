# Round 1 fixer-report

## Changes

- **F1** (fixed as proposed): `internal/beamcli/dobbypairanchor_test.go`.
  The `DobbyRuns` land-outcome test gained a board with open, done and gone entries and a pair removed from disk.
  It now asserts `EntryEnded` and `PairPresent` per slug.
  No test function was added; the test and the file header were renamed to say what they now pin.
- **F2** (fixed as proposed): `internal/dobbyengine/ended_test.go`, `TestTearDownEnded`.
  A row runs the loop with a never-delivering teardown and expects no `CheckBinary` call.
  A row with a nil `TearDownEnded` expects no teardown, no in-flight mark and no wait.
- **F3** (fixed as proposed): same file.
  The failing-teardown row runs 12 failures and asserts the wait and the time to the next attempt both sit at `endedTeardownCeiling`.
  The removed-pair row asserts `endedRetries` holds no entry for the slug.

Commits: the four `fix` commits on this branch, one pair per file.
F2 and F3 share a commit because both change `ended_test.go`.

A gate finding then flagged the wrapped line 2 of the `dobbypairanchor_test.go` header.
It is now two sentences, one per line, and the commit below records it.

## Self-check

- fix-code-correctness, LOW: ceiling comment and message overstated the failures read. Fixed as proposed.
- fix-consistency, LOW: test name stale after the added assertions. Fixed as proposed.
- fix-consistency, LOW: file header stale. Fixed as proposed.
- fix-proposal, MEDIUM: ceiling reached on the 10th failure, not the 11th. Fixed as proposed.
- fix-proposal, LOW: the ceiling assertion left `next` unchecked. Fixed with my own change: the loop now stops at 12 calls and one more tick reads the 12th result,
  so `next` is exactly one ceiling past the clock. Asserting right after the 13th call would be 10s off.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
