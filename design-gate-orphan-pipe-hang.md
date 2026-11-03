# A gate tier whose shell exits while a grandchild holds its pipe hangs forever

`gate-orphan-pipe-hang` · Task · bug, gate, process · running · Record-Write

A gate tier or build step whose shell exits while a grandchild still holds its stdout pipe never returns: `lym gate test` hangs forever, and neither `timeout_min` nor the tied group kill can end it.

## Seen

On the `cs` hub (`lym-csharp-LYMHUB`, Neve, 2026-11-03), the adopt check of `root` started 15:15:31 and ran tier0, `dotnet test LymCsharp.sln --filter "Category!=Slow"` in `lym-csharp/dotnet`.

- At 15:15:32 the `dotnet` process (pid 3374099) aborted, `Internal CLR error. (0x80131506)`, core dumped (`coredumpctl info 3374099`); `suite.log` holds only those three lines.
- The tier's shell (pid and pgid 3374098) exited and was reaped, but the six MSBuild worker nodes `dotnet test` had started (`MSBuild.dll /nodemode:1 /nodeReuse:false`, pgid 3374098, reparented to `systemd --user`) stayed alive and idle, each holding the write end of the tier's stdout pipe (`pipe:[435404635]` on fds 1, 2, 6, 7, 45).
- The check, `lym adopt check root` (pid 3373920), holds the read end on fd 10 and still reports `tier0 running; no tier done` well past the tier's `timeout_min: 10`.
  `slot-1.yaml` stays held for `lym gate test tier0`.

## Cause

`internal/gatecli/tierrun.go` and `internal/gatecli/build.go` set `cmd.Stdout`/`cmd.Stderr` to an `io.Writer` that is no `*os.File`, so `exec.Cmd.Wait` copies through a pipe and waits for EOF, and neither sets `cmd.WaitDelay`.
Once `Process.Wait` returns, Go's `watchCtx` stops watching the context, so the tier's or build's timeout no longer calls `cmd.Cancel`, the group kill in `runTiedChild` never runs, and `awaitGoroutines(nil)` blocks for as long as any holder of the pipe lives.
`internal/treadleengine/gate.go`, `internal/verifyrun/run.go`, `internal/gitexec/gitexec.go` and `internal/desktophook/window.go` already bound this read with `WaitDelay`; the gate's own tier and build children do not.
An MSBuild node outlives a crashed `dotnet` parent routinely, so any .NET hub can hit this.

## Expected

- A tier or build child whose shell has exited ends the run within a bounded delay, even while a grandchild holds the pipe: set `WaitDelay` on both commands and treat `exec.ErrWaitDelay` after an exit as that exit.
- After the shell exits, lym kills what is left of its process group, so orphaned workers do not hold the pipe or a gate slot.
- The tier records the shell's exit code (134 here) and fails, so the adopt check ends red and its fix loop runs, instead of reporting `running` with no end.
