# Settings shared by every session of a worktree come only from the heddle-rendered file; per-session ones stay per session

`claude-settings-model-effort` · Task · enhancement, claude, config, shuttle · running · Drift-Verify

## Seen (board tidy and orch, 2026-11-07)

Heddle's `claude-settings` target (`72390eb3`) renders `.claude/settings.local.json` in every hub worktree, triggered by fabric, so a session the operator starts by hand gets lym's settings.
Lym's own sessions also get a per-session `--settings` document that `internal/shuttleengine/claudeengine` builds at each launch (`settings.go`, `scope.go`).
Some of what that document sets is the same for every session of a worktree, such as `attribution` and `promptSuggestionEnabled: false`, so those settings are spread over two files.
That commit also left model and effort defaults to be added to the target later.

## Wanted (operator, 2026-11-07)

The point: what lym sets in Claude's settings is steered from one place per kind and can be read where it belongs.

- Every setting that is the same for every session of a worktree is rendered by heddle into `.claude/settings.local.json` and no longer set in the per-session document.
- Every setting that differs per session or role stays in the per-session `--settings` document: hooks that name one session's own paths, such as the events file, and anything bound to one role.
- Per-role differences in `enabledPlugins` and `permissions.deny` are checked against the operator's rule that every agent keeps the same tool set; a difference that rule removes goes, and a setting that is then the same for every session moves to heddle.
- The machine-wide model and effort defaults are rendered into the heddle file from the `agents.yaml` that `agent-skills-and-effort` builds, so each value has one source; per-role effort stays on the launch.
- Planned to follow the landing of `agent-skills-and-effort`, which changes the same launch code.
