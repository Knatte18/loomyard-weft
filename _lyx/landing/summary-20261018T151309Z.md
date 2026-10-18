# Tests: no short bound on a wait or timeout expected to succeed

Fixes #716.
Merge-In verifies failed in the `internal/shedverbs` integration tier with `loop … exited before it recorded itself`.
The cause was the wait-world fixture's 20 ms `LockGrace`: under a scheduling stall, the waiter's free-lock probe held the loop lock longer than the grace, so the detached loop gave up before it wrote its pid file.
Commit `743c11f0d` already moved that fixture onto the production grace.
This change removes the field it left without a setter, and fixes the other tests in the integration tier that put a short bound on a path expected to succeed.

## What changes

- `internal/shedverbs`: `LoopSpec.LockGrace` is gone, and `runLoop` waits `loopLockGrace` for the loop lock, so no fixture can shorten the grace again.
- `internal/preflightshed`: the two preflight cases over a free hub lock, the stale-stamp reconcile and the unparseable config in another pair, use `NewPreflight` with the default lock wait instead of 50 ms.
  The held-lock case keeps 50 ms, because the timeout is what it tests.
- `internal/lock`: the free-lock row of `TestAcquireWriteLockWithin` waits a minute instead of 50 ms; a free lock is taken on the first try, so the row costs no time.
- `internal/verifytree`: `TestVerify_SlotGate` gives the command one second instead of 300 ms to spawn its shell once the slot frees.
- `internal/githubclient`: the gh auth token timeout test allows two seconds over the shrunk timeout instead of 200 ms, still under the 5 s production timeout whose silent use it catches.

## Design decisions

- A test's wait or timeout is short only where its expiry is the behavior under test.
  Bounds whose expiry a test asserts, and "still blocked after X" checks, stay as they are: a stall makes those pass, never fail.
- `TestVerify_SlotGate` stays real-time.
  It proves the timeout does not count while the slot is held, so its hold must outlast the timeout; the hold stays at twice the timeout, which adds about 1.4 s to the integration tier.
- The lock unit test and the githubclient test are untagged, but the integration run executes them too, so they are in scope.

## Testing

The touched packages pass under `lyx gate test` in both the default and the integration tier.
