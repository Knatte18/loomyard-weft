# Python guard: state why it refuses and how to lift it

An agent in a lym-spawned session that reached for Python, as OCR work does, was refused without learning why or what to do next.
The refusal now says why, and names the way forward for work that needs Python.
This is a text change only; no new switch.

## What changes

- `internal/shellguard/rows.go`: `pythonRefusal` now gives the reason (what Python runs differs from machine to machine with installed packages, venvs and installations, and it is often very slow).
  For work that needs Python itself, it tells the agent to say so to the operator or its parent, who can set `deny_python: false` for the hub, and never to work round the guard.
  The system-prompt notice (`noticePythonDeny`) and the steer (`steerPythonDeny`) both build on it, and so does the Codex steer.
  `pythonRoutes` now scopes the jq/grep/awk parse routes to work that does not need Python.
- `internal/shuttleengine/template.yaml` and the `DenyPython` doc in `config.go`: state that `deny_python: false` removes both the hook and the system-prompt sentence, why the default is on, and the `lym config shuttle --layer default --set deny_python=false` command from the prime worktree.
- `lym config` help (`internal/configcli/configcli.go`): adds the same command, built from `shellguard.SwitchDenyPython`.
  To keep the long help under its byte cap, some existing sentences in `buildConfigLong` are shortened without changing their meaning.
- `internal/shellguard/rows_test.go`: `TestCheckTexts` also asserts the Codex python steer opens with the shared refusal.
- `internal/shuttleengine/claudeengine/testdata/settings-golden.json` is regenerated for the new notice text.

## Design decisions

- No Heddle line.
  Heddle renders to `CLAUDE.local.md`, which sessions lym did not start also read, while the guard exists only in lym-spawned sessions; the system prompt is the right place.
- The steer stays free of quotes and backslashes, which `rows.go` checks at init.
- The existing switch already drives the hook and the notice from one row, so no code path changes.

## To check by hand

- Not run by hand: a real lym-spawned session's refusal and system prompt reading as intended; the wording is covered by the tests and golden file only.
- No regression check ran for this task.
