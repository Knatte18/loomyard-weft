# One owner for child-process lifetime in internal/proc

`proc-child-lifetime` · Note · enhancement · 

Two process-lifetime implementations exist once `gate-test-scope` and `git-robustness` both land: `internal/proc` `ConfigureGroupKill` (process group, group SIGKILL on cancel, shared by `verifyrun` and `gitexec`) and `lyx gate test`'s own child lifetime (process group, group kill on a caught signal, Linux `Pdeathsig`, Windows kill-on-close job object).
The orch kept them apart on 2026-10-10 so the two parallel branches would merge without conflict.

Idea: move the `lyx gate` child-lifetime setup into `internal/proc` beside `ConfigureGroupKill`, so one package owns child-process lifetime, and consider whether `verifyrun` and `gitexec` should gain the parent-death signal and job object as well.

The package also lacks a `doc.go`; its file headers carry the description (PR-Gate review of PR #483).
