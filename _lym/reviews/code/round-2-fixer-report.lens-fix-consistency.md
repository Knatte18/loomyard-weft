```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix's change set for places that still disagree with it. A fresh search of Go, Markdown, YAML and JSON outside `_lym`, `.lym` and `.scratch` found nothing stale:
- Nothing refers to `SwitchAutonomous`, `guardInputs.autonomous`, or a tmux row limited to autonomous or non-interactive runs.
- `internal/shellguard/doc.go:65` is the only remaining match, and it states the new rule ("an interactive session carries it too").
- The old "refused in an autonomous run" steer wording is gone from the code and from the regenerated golden.
- The `shuttleengine/doc.go` and `codexengine/doc.go` search turned up no `DenyPublish` or `AllowRawTmux` mention, so there is no field list there to keep in step.

The engines agree with each other. Both Claude and Codex install the row unless `AllowRawTmux` is set. Both manifest tests expect the row on interactive specs and not on an exempt one. The sampler spec and its test pin `AllowRawTmux`.

- `grep -rIn` for stale `SwitchAutonomous`, `guardInputs.autonomous` and autonomous-or-interactive tmux wording across Go, Markdown, YAML and JSON: this checks the twins of the changed text, which my context did not hold.
- `grep -rn` for `AllowRawTmux|DenyPublish` in the shuttleengine and codexengine `doc.go` files: this checks whether a field list needed the new field, which my context did not hold.
