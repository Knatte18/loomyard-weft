# Python guard: say why it refuses and what to do when a task needs Python

An agent refused by the python guard learned neither the reason nor what to do when the task needs Python itself, as OCR processing does.
This is a text change only: the guard matches what it matched before, and no switch is added.

## What changed

- `pythonRefusal` in `internal/shellguard/rows.go`, which feeds the plain steer, the Codex steer and the notice, now states the reason: what Python runs differs from machine to machine, and it is often very slow.
  It also names the way forward: the agent says so to the operator or its parent, who can set `deny_python: false` for the hub, and the agent never works round the guard.
- The steer's parse routes now open with "otherwise", since they apply to work that does not need Python itself.
- `internal/shuttleengine/template.yaml` names the switch's effect on both the hook and the system-prompt sentence, the reason, and the command that sets it.
- `lym config`'s help names the same effect and the hub-wide command.
- The `settings-golden.json` fixture carries the new text, and `TestCheckTexts` and `TestConfigLong` pin the reason, the way forward and the command.

## Design decisions

- The hub-wide route is `lym config shuttle --layer default --set deny_python=false` from the prime.
  `shuttle` is a per-worktree module, so a bare `lym config shuttle --set` writes one worktree's file and would not reach the hub.
- The switch's effect is named in `lym config`'s help as well as the template.
  Under a hub the base file is sparse, so neither `lym config shuttle --print` nor the editor shows the template's comment.
- The help and the steer build the key name from `shellguard.SwitchDenyPython`, which adds an import of `shellguard` to `configcli`.
- `docs/research/tokens/startup-context-2026-11-05/classify.jq` still matches the old wording: it is a dated measurement, and it stays as it ran.
