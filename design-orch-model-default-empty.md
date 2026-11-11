# The orch session starts on the provider default model because orch.model and orch.effort default to empty

`orch-model-default-empty` · Task · bug, orch, config · running · Finalize

## Seen (operator, 2026-11-11)

On OSL-1046 the orch of the prime started on Sonnet.
The `orch` module's template holds `model: ""` and `effort: ""`, and `internal/orchcli/start.go` passes them as the session's `ModelChoice`, where empty means the provider default.
So the orch runs on whatever Claude Code's default is on that machine (a local `/model`, `~/.claude/settings.json` or the account's default), unlike every other agent in lym, whose model and effort come from a config default.

## Wanted

The orch's model and effort come from lym's config with a non-empty default in the template, so `lym s` and `lym beam start` start the orch on the configured model on every machine.
The template default is `model: opus` and `effort: medium` (operator, 2026-11-11, "for now").
The operator has already set both in this hub's default layer; the template default makes every hub and machine start the same way.
Check that every path that starts an orch session, `lym s` included, reads `orch.model` and `orch.effort`.
