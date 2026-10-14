# Card 7 — proc-process-listing

**Edit:**
- `internal/proc#TestProcessTreeReads`
- `internal/proc/doc.go`
- `internal/proc/proc_linux.go`
- `internal/proc/proc_windows.go`

**Create:**
- `internal/proc#Process`
- `internal/proc#ListProcesses`
- `internal/proc/process.go`

**Intent:**
The process reading behind the stale-binary listing (D7) lives beside `internal/proc`'s other `/proc` readers, and `tenderengine` is told it as a seam (card 9).

`Process`, in the new untagged file `process.go`, `package proc`, is one process as read:
`PID int`, `StartTime string` (the form `StartTime` returns), `Exe string` (the `/proc/<pid>/exe` link target as the kernel prints it, a ` (deleted)` suffix included), `Cwd string` (the `/proc/<pid>/cwd` link target, likewise) and `Argv []string` (the NUL-split `cmdline`).

`ListProcesses` is declared once per platform file:
- in `proc_linux.go` it lists every numeric entry of `/proc` and reads each one's start time, exe link, cwd link and command line.
  A process that ends between the listing and its reads, or whose links the caller may not read, as another user's, is skipped;
  an unreadable `/proc` is an error.
- in `proc_windows.go` it returns `errors.ErrUnsupported`, as `TreeCPU` does there.

`doc.go` names `ListProcesses` beside `TreeCPU` and `IsAncestor`, Linux only.

`TestProcessTreeReads`, which already starts a `sleep 30` child to pin the `/proc` readings, gains assertions on `ListProcesses`:
the child is listed with its pid, the same start time `StartTime` reads, an `Exe` whose base is `sleep`, the test's working directory as `Cwd`, and `Argv` `["sleep", "30"]`;
and the test process itself is listed with `Exe` equal to `os.Executable()`.

**ImpactSummary:** One new exported reader with its type in `internal/proc`; no existing function changes.

**Commit:** 7: proc: list processes with their exe and cwd links
