# Lym sets the Claude Code defaults it needs: no co-author, its own model and effort defaults

`machine-claude-defaults` · Note · enhancement, claude, config, shuttle

## Seen

The operator (2026-11-07) wants lym to set up, on the machine it runs on, the Claude Code settings lym needs to work as the operator wants:

1. No Claude co-author or session link in commits and pull requests; LoomYard has its own credit lines.
2. Model defaults that match lym's own, so no session or subagent runs at an effort lym did not choose, such as `xhigh`.

Today lym writes no `.claude/settings.json` into any worktree.
It writes a settings document per session and passes it with `--settings` (`internal/shuttleengine/claudeengine/settings.go`), and every such document already sets `attribution` to empty commit and pull-request texts (`780b1db4`), for run sessions and sessions opened in a hub alike.
A checked-in `.claude/settings.json` is the repository's own, which is why another machine's repositories differ.
A Claude session not started by lym, or lym sessions on a machine running an older lym, get none of this.

`~/.claude/settings.json` sets `effortLevel` per model under `modelSettings`, yet subagents with no `effort` ran at `xhigh` in `comment-cleanup`; whether a setting reaches a subagent's default is what `agent-skills-and-effort` checks.

## Idea

- Lym sessions: keep everything in the per-session settings document, which reaches every session lym launches without writing a file into the user's repository; add the model and effort defaults from lym's config there.
- Sessions outside lym: a lym verb the operator runs on a machine, such as `lym setup claude`, writes the same keys into `~/.claude/settings.json`, showing the diff and asking first; lym never edits the user's global settings unasked.
