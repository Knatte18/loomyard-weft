# Decision record — wait-world-lock-grace-flake

## Goal

Make the test tiers that verify runs deterministic against scheduling stalls: no test passes a short bound on a wait or timeout whose expiry it does not assert.

## Scope

- The wait-world fixture in `internal/shedverbs/loopwait_test.go` already runs on the production loop lock grace, since commit `743c11f0d` removed its 20 ms `LockGrace`.
  This task removes the `LoopSpec.LockGrace` field that commit left without a setter, so the loop lock grace is `loopLockGrace` for every caller and no fixture can shorten it again.
- The audit of the other fixtures fixes each short bound that sits on a success path:
  - `internal/preflightshed/preflight_integration_test.go`: the two cases over a free hub lock pass `LockWait: 50ms`; they take the default.
  - `internal/lock/lock_test.go`: the free-lock row of `TestAcquireWriteLockWithin` waits 50 ms for a lock nothing holds; it gets a wait no stall exhausts.
  - `internal/verifytree/verify_integration_test.go`: `TestVerify_SlotGate` gives the command 300 ms to spawn a shell and run once the slot frees; the timeout grows to one second.
  - `internal/githubclient/githubclient_test.go`: `TestRunGHAuthTokenSeam_HonoursGhAuthTokenTimeout` fails when the timed-out call takes over 210 ms; its slack grows to two seconds, still under the 5 s production timeout it guards against.
- Out of scope: bounds whose expiry the test asserts (a held lock that must time out, a hanging turn that must be skipped), and "still blocked after X" checks, which a stall makes pass rather than fail.

## Decisions

### Remove `LoopSpec.LockGrace`

Decision: delete the field and its zero-selects-default branch; `runLoop` waits `loopLockGrace` for the loop lock.
Rationale: nothing sets the field since the wait-world fixture dropped it, and a dead knob only invites a fixture to shorten the grace again.

### A short bound stays only where its expiry is the behavior under test

Decision: a test's wait or timeout is short only when the test asserts that it runs out; on a success path it takes the production default or a bound a scheduling stall cannot exhaust.
Rationale: the flake in issue #716 was a 20 ms grace on a path expected to succeed, and the same shape elsewhere fails the same way under load.

### Pay real time only where the bound is the assertion

Decision: `TestVerify_SlotGate` keeps its real-time hold of the slot, at twice the timeout, with the timeout raised to one second.
Rationale: the test proves the timeout does not count during the slot wait, so the hold must outlast the timeout in real time; one second gives the shell room under load for about a second more of suite time.

## Constraints

- `PATTERN-test-speed`: tests stay at their tier and parallel where they were; added real time is kept to the one test whose assertion needs it.
- `PATTERN-test-economy`: no test is added; the change edits existing rows and fixtures.
- `PATTERN-comment-line-breaks`: comments one sentence per line.

## Auto-mode assumptions

- "The integration tier" means every test `lyx gate test --tags integration ./...` runs, which includes untagged tests in tagged packages.
- The wait-world fixture fix named in the board entry is the already-landed `743c11f0d`, so this task does not redo it.

## Open risks

- The verifytree and githubclient bounds stay real-time; a stall longer than their new bounds still fails them, though far less often.

## Acceptance criteria

- `LoopSpec` has no `LockGrace` field and the loop lock wait is `loopLockGrace`.
- The wait-world, preflight, lock, verifytree and githubclient tests pass in their tiers with the new bounds.
- A held-lock case still times out on its short bound.
