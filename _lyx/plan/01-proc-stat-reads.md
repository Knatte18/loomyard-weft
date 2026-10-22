# Card 1 — proc reads every process's parent, group, start time and state, and signals a pid or a group

**Create:**
- `internal/proc#StatEntry`
- `internal/proc#ListStats`
- `internal/proc#SignalProcess`
- `internal/proc#SignalGroup`
- `internal/proc#TestStatReadsUnsupportedOnWindows`

**Edit:**
- `internal/proc#TestProcessTreeReads`
- `internal/proc/doc.go`

**Intent:**
`internal/runproc`'s end sequence (D3) needs, for every process at once, its parent pid, its process-group id, its start time and whether it is a zombie, and needs to send SIGTERM or SIGKILL to one pid or to one group.
`proc.ListProcesses` skips every process whose exe, cwd or cmdline link the caller cannot read and carries no parent or group, so it cannot serve; this card adds a stat-only listing beside it.

`StatEntry`, declared in `internal/proc/process.go` beside `Process`, holds `PID`, `PPID`, `PGID`, `StartTime` (the form `StartTime` returns) and `Zombie` (field 3 of `/proc/<pid>/stat` reads `Z`).
`ListStats` on Linux, in `proc_linux.go`, reads every `/proc/<pid>/stat` through the existing `statFieldsAfterName` (state is entry 0, parent entry 1, group entry 2, start time entry 19), skips a process that ends between the listing and its read, and returns an error only for an unreadable `/proc`.
On Windows, in `proc_windows.go`, it returns `errors.ErrUnsupported`, as `ListProcesses` does.

`SignalProcess` sends SIGTERM, or SIGKILL when `kill` is set, to the pid; `SignalGroup` sends it to `-pgid`.
Each returns the raw syscall error, so a caller tells `syscall.ESRCH` (already gone) from a real failure.
On Windows `SignalProcess` with `kill` set calls `KillPID`, without it returns `errors.ErrUnsupported`, and `SignalGroup` always returns `ErrNoProcessGroup`.
proc stays a leaf of the standard library: nothing here logs.

`TestProcessTreeReads`, which already starts a `sleep` child, gains assertions: `ListStats` lists the child with `PPID` equal to `os.Getpid()`, a non-empty `StartTime` equal to `StartTime(childPID)`, and `Zombie` false;
after `SignalProcess(childPID, false)` and before the child is waited for, a bounded poll of `ListStats` (every 10ms, up to five seconds, failing the test at the bound) sees it with `Zombie` true, since the signal is delivered asynchronously;
`SignalProcess` on the now-reaped pid returns an error wrapping `syscall.ESRCH`;
`SignalGroup` on the reaped child's pid as a group id, which no process now carries, returns an error wrapping `syscall.ESRCH`.
`TestStatReadsUnsupportedOnWindows`, in `proc_windows_test.go`, asserts `ListStats` returns `errors.ErrUnsupported`, `SignalProcess(pid, false)` returns `errors.ErrUnsupported` and `SignalGroup` returns `ErrNoProcessGroup`.
The package `doc.go` names `ListStats`, `SignalProcess` and `SignalGroup` beside `ListProcesses`, Linux-only like it.

**ImpactSummary:** additive: three new functions and one new type in `internal/proc`, no existing caller changes.

**Commit:** 1: proc reads parents, groups and zombie state, and signals a pid or a group
