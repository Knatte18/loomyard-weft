# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): README prime-owner line is now asserted at each production render site.
  - `internal/boardengine/board_test.go`: new `TestPrimeOwnerReadAndRendered` seeds a recorded prime owner, writes through `UpsertTask` and asserts the rendered README states it.
  - `internal/boardengine/ownerset_integration_test.go`: the `SetOwners` outcome rows that write assert the README holds the prime-owner line for `wantPrime`.
  - `internal/boardengine/replay_test.go`: each replay row that is not empty asserts the rendered README holds the prime-owner line for `wantPrime`.
  - Fixed as a new test in `board_test.go` rather than an assertion in an existing board-write test, since no existing test seeds a prime owner.
- F2 (fixed with my own change): the proposed test of the absent-directory branch of `Board.PrimeOwner` cannot fail, since `Store.Load` already reads an absent directory as empty with no error (`internal/boardengine/store.go:159`).
  The branch was redundant, so `internal/boardengine/board.go` drops it and `TestPrimeOwnerReadAndRendered` pins the absent-directory result.
  The proposed CLI row was not added; the engine test pins the same contract.
- F3 (fixed as proposed): `internal/boardcli/cli.go` find help now says it prints the same tasks array as `lyx board list`, in an envelope without `prime_owner`.

Commits: `8218ea5f` (F3), `9c1e5204` (F1, F2 test), and the follow-up that drops the branch.
`lyx gate test` for `./internal/boardengine` and `./internal/boardcli` is green at tier 0, and tier 1 for `./internal/boardengine` is green.

## Self-check

- fix-code-correctness, MEDIUM: the absent-directory assertion passes with the branch deleted. Fixed with my own change: the redundant branch was removed.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
