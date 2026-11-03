# gatecli: end a tier whose shell exits while an orphan holds its pipe

`lym gate test` hung forever when a tier's or the build's shell exited while a grandchild still held its stdout pipe.
Seen on a .NET hub: `dotnet test` crashed, and the MSBuild worker nodes it had started outlived it and kept the pipe open.
`exec.Cmd.Wait` waits for EOF on that pipe, and once the shell is reaped the context watch stops, so neither `timeout_min` nor the tied group kill could end the run.
The tier stayed `running`, its gate slot stayed held, and the adopt check never went red.

## What changes

- `runTiedChild` in `internal/gatecli/child.go` sets `cmd.WaitDelay`, so a tier or build reads its output for at most that delay after the shell exits.
  Production uses 10 seconds (`defaultPipeWaitDelay`).
- After the wait, `runTiedChild` kills what is left of the shell's process group through `proc.KillTree`, so orphaned workers stop holding the pipe or running inside the slots.
  On Windows nothing is killed here, because closing the child's kill-on-close job already ended the tree.
- The tier or build records the shell's own exit code, 134 in the seen case, and fails, so the adopt check ends red and its fix loop runs.
- `proc.KillTree`'s doc comment no longer claims the start-time check leaves a reused group id alone; it leaves an emptied group alone.
- `doc.go` gains a paragraph under "Child lifetime" naming the delay and its bound.

## Design decisions

- `exec.ErrWaitDelay` is mapped to success only when the shell exited zero, because Wait reports the delay error only after a zero exit; a failing exit keeps its own error.
  A zero exit cut short by the delay logs a warning and returns nil.
- The delay is a `newCommand` parameter (`pipeWaitDelay`, carried on `testRequest`), like the timeout unit and the pre-tier ceiling, so tests shorten it.
  `RunGate` passes the default.
- Bound: a leftover that left the shell's process group, such as one that started its own session, is not killed, and only the delay bounds its hold on the pipe.

## Tests

`TestGateTest_Scenario` in `internal/gatecli/test_integration_test.go` gains an orphan stand-in (`GATECLI_TEST_ORPHAN`) and three scenarios:

- a tier whose shell exits non-zero (134) while an orphan holds its output;
- a tier whose shell exits zero in the same situation;
- the shared build whose shell exits 1 in the same situation.

Each asserts the shell's exit code, the run record's outcome, that the orphan is gone and that the slots are free.
These are Tier 2 tests, which need the integration build tag.

## To check by hand

- The shared build's bounded end is covered only by the stand-in scenario above; confirm it exercises the `build.go` path rather than the tier path.
- The zero-exit `ErrWaitDelay` row, mapped to a pass, is covered only by the stand-in scenario; confirm a real tool that exits zero and leaves a pipe holder passes.
- Not run: `lym gate test` against a real .NET hub with a crashing `dotnet test` and live MSBuild nodes.
