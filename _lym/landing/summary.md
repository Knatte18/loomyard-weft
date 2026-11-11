# orch: default the session model to opus and effort to medium

The `orch` module's config template held `model: ""` and `effort: ""`.
`internal/orchcli/start.go` passes both as the session's `ModelChoice`, where empty means the provider default.
The orch therefore ran on whatever Claude Code defaulted to on that machine: a local `/model`, `~/.claude/settings.json`, or the account default.
On OSL-1046 the prime's orch started on Sonnet.
Every other agent in lym takes its model and effort from a config default.

The template in `internal/orchengine/template.yaml` now sets `model: opus` and `effort: medium`.
`lym s` and `lym beam start` start the orch on the same model and effort on every hub and machine unless a config layer overrides them.

## Design decisions

- The values are the operator's choice "for now".
  They live only in the template, so changing them later is a one-line edit.
- The key comments now say an explicit empty value still selects the provider default.
  The override stays available to a hub that wants it.
- `ConfigTemplate` in `internal/orchengine/template.go` has its doc comment updated to match.
- `TestLoadConfig_TemplateResolvesWithNoFile` now expects `opus` and `medium` from the template.

## To check by hand

- The change edits no code path.
  The task also asked that every path starting an orch session, `lym s` included, reads `orch.model` and `orch.effort`.
  That was not re-verified in this diff, so confirm that `lym s` and `lym beam start` both load the orch config and pass both values.
- Start an orch with `lym s` on a machine with no local model override and confirm it runs on opus at medium effort.
