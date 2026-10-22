# Card 2 — a cancelled verify command gets SIGTERM on its group, then SIGKILL

**Create:**
- `internal/proc#ConfigureGroupTerminate`
- `internal/proc#configurePlatformGroupTerminate`

**Edit:**
- `internal/verifyrun#run`
- `internal/verifyrun#Run`
- `internal/verifyrun#TestRun_ExpiryKillsBackgroundedDescendant`
- `internal/verifyrun#TestRun_Cancellation`
- `internal/proc/doc.go`

**Uses:**
- `internal/proc#ConfigureGroupKill`
- `internal/proc#ErrNoProcessGroup`

**Intent:**
D4's SIGTERM-first cancel, a new cancel used only by `verifyrun`; `proc.ConfigureGroupKill` keeps its immediate group SIGKILL for `gitexec` and the `lyxbin` test kit, and its code is not touched.

`ConfigureGroupTerminate` in `internal/proc/procgroup.go` delegates to `configurePlatformGroupTerminate`, defined once in each of `procgroup_linux.go` and `procgroup_windows.go`, beside `configurePlatformGroupKill`.
On Linux it starts `cmd` in its own process group (`Setpgid`), and sets `cmd.Cancel` to send SIGTERM to the whole group (`syscall.Kill(-pid, SIGTERM)`), report it, and start a timer of its own that sends SIGKILL to the same group after `killAfter` and reports that; a cancel whose SIGTERM was sent returns nil, so `exec` waits for the shell as usual and its own `WaitDelay` still bounds a lingering pipe holder.
The SIGKILL is sent whether or not the shell has already exited, and never through `exec.Cmd.WaitDelay`, which kills the shell alone and only while `Wait` has not returned.
`awaitGone` is called by the caller once `cmd.Wait` (or `Run`) has returned: when the cancel never ran it returns at once; otherwise it polls the group with `syscall.Kill(-pid, 0)` every 50 ms and returns once that answers `ESRCH` (the group is empty), stopping the timer so no late SIGKILL can reach a group that reused the pid, or once the timer has sent its SIGKILL.
A failed group SIGTERM is a fail-loud wiring guard in `contracts/specs/refusal-spec.md`'s sense: `cmd.Cancel` runs only while the unreaped shell keeps its group alive, so no state a run produces reaches it; the cancel reports it through `report` with its error and returns that error from `cmd.Cancel`, starting no timer, and `awaitGone` then returns at once; it takes no reaching test.
On Windows the cancel kills the shell alone and reports `ErrNoProcessGroup`, and `awaitGone` returns at once; the `verify-watchdog` board note owns that gap.
`report` is how the caller logs, since `proc` cannot log.

`verifyrun.run` replaces its `proc.ConfigureGroupKill` call with `ConfigureGroupTerminate(cmd, delay, report)`, logging each report at Info (`sending SIGTERM` / `sending SIGKILL after the wait delay`) and a failed or group-less signal at Warn as today, and calls `awaitGone` after `cmd.Run` returns and before it classifies a cancelled run, so `Run` returns only once no group member is left or the SIGKILL is sent.
`Run`'s doc says a cancel sends SIGTERM to the group, then SIGKILL after the wait delay, and returns only after the group is gone or that SIGKILL is sent.
Every `verifyrun.Run` caller gets this: `verifytree.Verify`, `cardverify`'s card `**Verify:**` rerun, and batten's `deploy` and `Command` rows, which act on their timeout only once `Run` returns.
The SIGTERM reaches `lyx gate test`, whose `terminatingSignalSet` handler kills its tied `go test` tree and releases its slot.

`TestRun_ExpiryKillsBackgroundedDescendant` becomes table-driven over a backgrounded descendant of the shell under an expiring context with a short delay: a descendant that exits on SIGTERM writes a marker from its `trap` and is gone when `run` returns; a descendant whose `trap '' TERM` ignores the SIGTERM is gone at most a few polls after `run` returns, and `run` took at least the delay.
`TestRun_Cancellation` keeps its bound and adds that the run's log carries the SIGTERM Info line.
`doc.go` names `ConfigureGroupTerminate` beside the other primitives.

**ImpactSummary:** one new proc primitive in its two platform files; `verifyrun.Run`'s cancel changes for its three production callers (verifytree, cardverify, battencli land), whose timeouts now SIGTERM first; gitexec and lyxbin keep `ConfigureGroupKill`.
