# Lym sessions take every static Claude setting from the heddle-rendered file, model and effort defaults included

`claude-settings-model-effort` · Note · enhancement, claude, config, shuttle

## Seen (board tidy and orch, 2026-11-07)

Heddle's `claude-settings` target (`72390eb3`) renders `.claude/settings.local.json` in every hub worktree, so a session the operator starts by hand gets lym's settings.
Lym's own sessions still get the same static settings a second time through the `--settings` document `internal/shuttleengine/claudeengine` writes per session (`settings.go`, `scope.go`): `attribution`, `promptSuggestionEnabled`, and per role `enabledPlugins` and a `permissions.deny`.
That commit also left model and effort defaults to be added to the target later.

## Wanted (operator, 2026-11-07)

- The heddle-rendered file is the one source of every setting that is the same for each session of a hub; lym's per-session `--settings` document stops carrying any of them.
  `promptSuggestionEnabled: false` and every other static key the per-session document sets today move to shipped heddle entries.
- The `--settings` document keeps only what cannot be the same in every worktree: hooks that name a run's own paths (the events file, creel's flag directory) and anything else bound to one session.
- Per-role differences in `enabledPlugins` and `permissions.deny` are checked against the operator's rule that every agent keeps the same tool set; a difference that rule removes goes, and one that must stay is named in the landing with why.
- The machine-wide model and effort defaults are rendered into the target from the `agents.yaml` that `agent-skills-and-effort` builds, so each value has one source; per-role effort stays on the launch.
- Planned to follow the landing of `agent-skills-and-effort`, which changes the same launch code.
