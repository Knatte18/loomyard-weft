# Windows verify timeout kills the whole process tree

`verify-watchdog` · Note · enhancement, shed

From run-recovery PR-Gate (2026-10-07): run-recovery gave `verifytree` a 60-minute timeout and kills the process group on Unix.
Still open from #420: on Windows the timeout kills only the shell, not the process tree it started.
The step-level watchdog for a `lyx shed step` that never returns moved into task `driver-context` (2026-10-10).

## Was `proc-child-lifetime`

Status (triage, 2026-10-12): the gate move is done; `lyx gate test` starts its child through `proc.StartTied` (`internal/gatecli/child.go:37`, f5b5218f7).
Left: `internal/proc` still has no `doc.go`, and whether `verifyrun` and `gitexec` should gain the parent-death signal and job object is still open.

Two process-lifetime implementations exist once `gate-test-scope` and `git-robustness` both land: `internal/proc` `ConfigureGroupKill` (process group, group SIGKILL on cancel, shared by `verifyrun` and `gitexec`) and `lyx gate test`'s own child lifetime (process group, group kill on a caught signal, Linux `Pdeathsig`, Windows kill-on-close job object).
The orch kept them apart on 2026-10-10 so the two parallel branches would merge without conflict.

Idea: move the `lyx gate` child-lifetime setup into `internal/proc` beside `ConfigureGroupKill`, so one package owns child-process lifetime, and consider whether `verifyrun` and `gitexec` should gain the parent-death signal and job object as well.

The package also lacks a `doc.go`; its file headers carry the description (PR-Gate review of PR #483).
