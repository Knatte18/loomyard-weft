# The python guard refuses without saying why or what to do when a task needs Python

`python-guard-gives-no-reason` · Task · enhancement, shuttle · running · Drift-Expect

## Seen

In another repository on lym, with OCR processing, agents reach for Python and are refused by the python guard without learning why (operator, 2026-11-06).

The guard's texts sit in `internal/shellguard/rows.go`: `noticePythonDeny` goes into the system prompt of a lym-spawned session, and `steerPythonDeny` is the refusal.
Both say Python is refused, and the steer gives routes for parsing and editing only.
Neither says why, what to do when the task needs Python itself, as OCR does, or that the guard can be switched off.

## What already holds

One config already decides both the block and the line telling agents not to use Python: `deny_python` in `shuttle.yaml`, on by default.
The hook and the system-prompt sentence are built from the same row, `buildDenyNotice` and `buildSettings` in `internal/shuttleengine/claudeengine/settings.go`, so `deny_python: false` removes both.
No Heddle line is needed: Heddle renders to `CLAUDE.local.md`, which a session lym did not start also reads, while the guard exists only in lym-spawned sessions, so the system prompt is the right place.

## The reason (operator, 2026-11-06)

Python is unreliable from machine to machine: what runs depends on what the machine has installed, its venvs, packages and installations.
It is also often very slow.

## Wanted

This is a text change only, with no new switch.

- The refusal and the notice state that reason, and name the way forward for work that needs Python: the agent says so to the operator or its parent, who can set `deny_python: false` for the hub, and the agent never works round the guard.
- `lym config shuttle` and the template name the switch's effect on both the hook and the notice.

The steer must stay free of quotes and backslashes, which `rows.go` checks at init, and the Codex steer changes with it.
