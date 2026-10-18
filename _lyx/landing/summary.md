# Remove LoopSpec.LockGrace; widen short success-path test bounds

Verifies failed intermittently in the `internal/shedverbs` integration tier with `loop … exited before it recorded itself`.
The cause was a wait-world fixture that shortened the loop lock grace to 20 ms, which a scheduling stall exhausts on a path that is expected to succeed.
The fixture already runs on the production grace; this change closes the knob and fixes the same shape elsewhere.

Closes #716

## Changes

- `internal/shedverbs/loop.go`: delete `LoopSpec.LockGrace` and its zero-selects-default branch.
  Nothing sets the field any more, so `runLoop` waits `loopLockGrace` for the loop lock and no fixture can shorten it again.
- `internal/preflightshed/preflight_integration_test.go`: the two cases over a free hub lock use `NewPreflight` and the default lock wait instead of `LockWait: 50ms`.
- `internal/lock/lock_test.go`: the free-lock row of `TestAcquireWriteLockWithin` waits one minute instead of 50 ms for a lock nothing holds.
- `internal/verifytree/verify_integration_test.go`: `TestVerify_SlotGate` raises its command timeout from 300 ms to one second, since it bounds a shell spawn after the slot frees.
- `internal/githubclient/githubclient_test.go`: the slack in `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout` grows from 200 ms to two seconds, still well under the ~5 s production wait the check guards against.

## Design decisions

- A test's wait or timeout stays short only when the test asserts that it runs out.
  Held-lock cases that must time out, and "still blocked after X" checks, keep their short bounds; a stall makes the latter pass rather than fail.
- `TestVerify_SlotGate` keeps its real-time slot hold, which must outlast the timeout, so the larger timeout adds about a second of suite time.
  That is the one place real time is paid, because the hold is the assertion.

## To check

- The `verifytree` and `githubclient` bounds are still real time, so a stall longer than their new bounds can still fail them, far less often.
- Not run by hand: `lyx gate test ./...` and `lyx gate test --tags integration ./...` across the touched packages.
