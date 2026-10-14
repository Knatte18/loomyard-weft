---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedcli/attach_unix.go:24
    summary: handOver passes the unresolved tmux name to syscall.Exec, which does no PATH lookup, so every real attach fails with no such file or directory
    fix: In handOver in internal/reedcli/attach_unix.go, resolve the binary with exec.LookPath(tmux) before the exec (add the os/exec import) and pass the resolved path as both the path argument and argv[0] of syscall.Exec; when LookPath fails, return exit code 1 and the error. Keep the testing.Testing() branch running runAttachChild with the original name. After the fix the five failing reedcli smoke tests (the two TestSmokeRepaintCandidateMeasurement candidates, TestSmokeDotFill/FloorIsCleanOnASettledAttach, TestSmokeColdStart/ColdAttachBootsThenFailsOnTheTerminalHandoverNotOnNoSession and TestSmokeLifecycle/AttachRendersInsideHarnessPane) must pass under lyx gate test --tags tmux ./internal/reedcli.
---

### [BLOCKING] handOver execs an unresolved tmux name

**Location:** internal/reedcli/attach_unix.go:24

**Issue:** `handOver` calls `syscall.Exec(tmux, ...)` with `tmux` from `reedengine.Engine.TmuxPath`.
`TmuxPath` returns `e.cfg.Tmux` unchanged (`internal/reedengine/lock.go:76`), and the default is the bare name `tmux`.
`syscall.Exec` takes a path and never searches `PATH`, so outside a test binary the exec fails with `no such file or directory`.
Before card 11 the child went through `exec.Command`, which does the lookup.
The Publish verify log `.lyx/verify/verify-7.log` shows the failure in every failing test: `reedcli: tmux attach handover failed tmux=tmux err="no such file or directory"` followed by `ATTACH-EXIT:1`.
The five named tests all run the built `lyx` binary, where `testing.Testing()` is false, so they reach the exec branch:
`TestSmokeRepaintCandidateMeasurement/Candidate1`, `TestSmokeRepaintCandidateMeasurement/Candidate2`, `TestSmokeDotFill/FloorIsCleanOnASettledAttach` and `TestSmokeLifecycle/AttachRendersInsideHarnessPane` never see their marker in the pane.
`TestSmokeColdStart/ColdAttachBootsThenFailsOnTheTerminalHandoverNotOnNoSession` finds no `not a terminal` text, because tmux never starts.
The five share this one cause, so one fix clears all of them.
The merge-in commit `a62ca297a` touches nothing in `internal/reedcli` or `internal/reedengine`, so the parent's changes are not the cause.
The in-process unit tests miss it because the `testing.Testing()` branch bypasses the exec path; the smoke tests are the only coverage of it, which is why they caught it.
The same `exec.LookPath` followed by `filepath.Abs` idiom already exists at `internal/reedengine/bindings.go:105`.
