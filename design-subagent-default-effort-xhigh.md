# Subagents spawned without an effort run at xhigh, not the medium settings name

`subagent-default-effort-xhigh` · Note · bug, shuttle, claude, stencil

## Seen

In `comment-cleanup` (2026-11-07), Haiku 5.5 workers spawned through the Agent tool with no `effort` ran at `xhigh`: about 36,000 transcript records carry `"effort":"xhigh"`.
Workers spawned with `effort: medium` ran at medium.
`~/.claude/settings.json` sets `effortLevel` medium for `claude-haiku-5-5` under `modelSettings`, and the operator never sets xhigh, so a subagent's default effort ignores that setting.
lym passes no effort to a subagent: the writer and the leads spawned theirs from their own briefs.

## Idea

Every stencil or brief that tells an agent to spawn subagents has it pass `effort` explicitly, and lym's own subagent spawns do the same.
Check whether the default is Claude Code's for subagents, and whether a setting can pin it.
