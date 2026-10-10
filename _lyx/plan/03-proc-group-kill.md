# Card 3 — Process-group kill moves from verifyrun to proc

**Create:**
- `internal/proc#ConfigureGroupKill`
- `internal/proc#ErrNoProcessGroup`

**Edit:**
- `internal/verifyrun/run.go`

**Delete:**
- `internal/verifyrun/procgroup_unix.go`
- `internal/verifyrun/procgroup_windows.go`

**Intent:** Decision D3 needs gitexec to kill a timed-out git's process group the way verifyrun kills a verify shell's, so the setup moves to `internal/proc` and both share one implementation.
`internal/logger` imports `internal/proc` (through `lyxcwd` and `gitexec` as well), so `proc` cannot log; `ConfigureGroupKill` hands each kill's outcome to the caller's `report` callback instead, and the caller logs.

`ConfigureGroupKill` lives in `procgroup_linux.go` and `procgroup_windows.go`, following `proc`'s own per-platform file naming:

- Linux: it sets `cmd.SysProcAttr` to `Setpgid: true` and `cmd.Cancel` to SIGKILL the group (`syscall.Kill(-pid, SIGKILL)`); when the group kill fails it kills the process alone and calls `report(pid, err)` with the group kill's error, and on success it calls `report(pid, nil)`.
- Windows: `cmd.Cancel` kills the process alone and calls `report(pid, ErrNoProcessGroup)`, since there is no process group to signal there.

`ErrNoProcessGroup` is declared once, in a platform-neutral file of `proc`.
`verifyrun.run` calls `proc.ConfigureGroupKill` with a `report` closure that logs today's three messages through `internal/logger` by outcome (group killed, group kill failed and shell killed alone, no process group so descendants may outlive the shell), and the two `procgroup_*.go` files are deleted.
`run.go`'s package doc, which says `verifyrun` imports only the standard library and `internal/logger`, names `internal/proc` as well.
Behavior is unchanged; `verifyrun`'s `TestRun_ExpiryKillsBackgroundedDescendant` and `TestRun_Cancellation` cover it, and card 4's timeout test exercises the group kill from gitexec.

**ImpactSummary:** `verifyrun.run`'s kill setup moves behind `proc.ConfigureGroupKill` with the same signals and log lines; no other caller changes.

**Commit:** 3: proc: one process-group kill shared by verifyrun and gitexec
