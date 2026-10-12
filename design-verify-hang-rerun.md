# Verify gate re-ran a hung test with no fix in between

`verify-hang-rerun` · Note · bug, shed, webster

## Seen (2026-10-12)

In `board-field-drop`'s Webster verify gate, `TestConcurrentReadsDuringUpserts` spun at about 600% CPU (a livelock in new code), so the gate ran for minutes and loaded the machine.
The orch stopped the test binary with SIGQUIT after about 7 minutes, and verify-1 failed with the goroutine stacks.
Two seconds later verify-2 started on the same code, with no fix in between, and hung the same way until it was stopped too.
The test binary's own timeout is 10 minutes, so a hang costs at least that, at full CPU, per attempt.

## Want

- A gate does not re-run a step that hung or was killed by a timeout or signal: a hang is a failure that goes to the fixer, not a flake.
  Which code started verify-2 (a flake retry in the gate, or the Webster loop) is the first thing to find.
- A gate step that keeps the CPU busy past a budget far below the 10-minute test timeout is stopped with SIGQUIT, so the stacks reach the log early.
