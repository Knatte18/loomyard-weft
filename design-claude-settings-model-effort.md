# Model and effort defaults on the claude-settings heddle target

`claude-settings-model-effort` · Note · enhancement, claude, config

## Seen (board tidy and orch, 2026-11-07)

Heddle's `claude-settings` target (`72390eb3`) renders `.claude/settings.local.json` in every hub worktree, and left model and effort defaults to be added to it later.
`agent-skills-and-effort` (running when this note was written) builds effort per role and subagent from `agents.yaml`; its scope was not widened to the new target.

## Wanted

- Once `agent-skills-and-effort` lands, decide whether the machine-wide model and effort defaults belong as entries on the `claude-settings` target, beside or in place of what `agents.yaml` sets, so one source holds each value.
- Planned to follow that landing; the operator agreed on 2026-11-07 to take it as its own entry rather than in the running task.
