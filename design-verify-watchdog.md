# Windows verify timeout kills the whole process tree

`verify-watchdog` · Note · enhancement, shed

From run-recovery PR-Gate (2026-10-07): run-recovery gave `verifytree` a 60-minute timeout and kills the process group on Unix.
Still open from #420: on Windows the timeout kills only the shell, not the process tree it started.

## Status (board triage, 2026-10-15)

The child-lifetime part of this note has landed: `internal/proc` owns child-process lifetime (`Tied`), has its `doc.go`, and `gatecli`, `verifyrun` and `gitexec` all use it.
The step-level watchdog for a `lyx shed step` that never returns moved into task `driver-context` (2026-10-10).

Left: on Windows a timeout still kills only the started process, not its tree (`procgroup_windows.go`).
