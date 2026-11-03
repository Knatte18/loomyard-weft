# A gate tier whose shell exits ends even while an orphan holds its pipe

`lym gate test` hung forever when a tier's or the build's shell exited while a process it had started still held the output pipe.
On a .NET hub this happens whenever `dotnet test` crashes and leaves MSBuild worker nodes alive: the adopt check then reported `tier0 running` past `timeout_min` and kept its gate slot.

## Cause

The tier and build commands write to an `io.Writer` that is no `*os.File`, so `exec.Cmd.Wait` copies output through a pipe and waits for EOF.
Neither command set `WaitDelay`.
Once the shell itself exits, Go stops watching the context, so the timeout never fires `cmd.Cancel` and the group kill never runs; `Wait` blocks for as long as any holder of the pipe lives.

## Change

`runTiedChild` in `internal/gatecli/child.go`, which both the tier run and the build step go through, now:

- sets `cmd.WaitDelay` to a pipe wait delay, 10 seconds in production, so the output read ends that long after the shell exits;
- returns nil when `Wait` reports `exec.ErrWaitDelay` after the exit, because Go reports that error only after a zero exit, and a failing exit keeps its own `*exec.ExitError`, so the run records the shell's exit code (134 in the reported case) and fails;
- kills what is left of the shell's process group once the shell exits, through `proc.KillTree` on the tied child's record, whose start-time check leaves a reused group id alone.
  On Windows, closing the child's kill-on-close job in `Tied.Wait` already ends the tree.

The delay is a new `newCommand` parameter and `testRequest` field, `pipeWaitDelay`, beside the other injected durations, so a test shortens it.
`RunGate` passes the production value.

The package doc's Child lifetime section describes the new behavior and its bound: a leftover that started a session of its own leaves the shell's process group and is not killed, so only the delay bounds its hold on the pipe.

## Design decisions

- The leftover kill runs after every exit, not only after the wait delay ran out, so leftovers that closed their output but kept running inside the slots are ended too, as the entry's Expected section asks.
- The delay is injected rather than a package constant read directly, per `PATTERN-test-speed`'s injectable-time rule.

## Tests

`TestGateTest_Scenario` (tier1) gains a subtest: the stand-in tier command starts a sleeper that inherits its stdout and then exits 134.
It asserts that the call exits 134, the run record says failed with exit code 134 and no timeout, the sleeper is killed, and the slots are free.
The subtest runs with a 100 ms pipe wait delay.
