## Changes

- F1 (LOW), fixed as proposed, commit `ae9a9c7e`: new `internal/reedengine/proctree_linux_test.go` (tier0, Linux-only by suffix) with `TestClockTicksFromAuxvReadsHostVector`.
  It calls `unix.Auxv()`, finds the key-17 entry, requires it positive, and asserts `clockTicksFromAuxv` returns the same value.
  `lyx gate test ./internal/reedengine` is green.

## Self-check

- fix-code-correctness, LOW: on a 100 Hz host a wrong key constant falls back to 100 and the host test still passes. Disputed, premise false (see below).
- fix-proposal, LOW: same case as above, duplicate. Disputed, premise false.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- Ground `premise false`, forks' LOW on the 100 Hz host: the claim that a wrong `auxvClockTicksKey` goes undetected holds only for the host test alone.
  `TestClockTicksFromAuxv` row `DeclaredRate` (`{6, 4096}, {17, 250}, {0, 0}` wanting 250) in `proctree_test.go` fails when the constant is not 17, on every host.
  The host test adds the check that a real kernel vector carries key 17 at all, which is the coverage the review asked for.

## Deferred

None.
