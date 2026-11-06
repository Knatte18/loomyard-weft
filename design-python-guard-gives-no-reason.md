# The python guard refuses without saying why or what to do when a task needs Python

`python-guard-gives-no-reason` · Note · enhancement, shuttle

## Seen

In another repository on lym, with OCR processing, agents reach for Python and are refused by the python guard without learning why (operator, 2026-11-06).

The guard's texts sit in `internal/shellguard/rows.go`: `noticePythonDeny` goes into the system prompt of a lym-spawned session, and `steerPythonDeny` is the refusal.
No Heddle entry states the rule, so nothing reaches `CLAUDE.local.md`.
Both say Python is refused, and the steer gives routes for parsing and editing only.
Neither says why, what to do when the task needs Python itself, as OCR does, or that `deny_python` in `shuttle.yaml` exists for such a repository.

## Idea

The refusal states why Python is refused and gives a way forward for work that needs it: the agent says so to the operator or its parent, who can set `deny_python: false` for the hub, and the agent never works round the guard.
The steer must stay free of quotes and backslashes, which `rows.go` checks at init, and the Codex steer changes with it.
