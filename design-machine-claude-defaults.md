# Heddle renders .claude/settings.local.json: no co-author, lym's model and effort defaults

`machine-claude-defaults` · Note · enhancement, claude, config, heddle, fabric

## Seen

The operator (2026-11-07) wants lym to set up, on the machine it runs on, the Claude Code settings lym needs to work as the operator wants:

1. No Claude co-author or session link in commits and pull requests; LoomYard has its own credit lines.
2. Model defaults that match lym's own, so no session or subagent runs at an effort lym did not choose, such as `xhigh`.

Today lym writes no `.claude/settings.json` into any worktree.
It writes a settings document per session and passes it with `--settings` (`internal/shuttleengine/claudeengine/settings.go`), and every such document already sets `attribution` to empty commit and pull-request texts (`780b1db4`), for run sessions and sessions opened in a hub alike.
A checked-in `.claude/settings.json` is the repository's own, which is why another machine's repositories differ.
A Claude session not started by lym, or lym sessions on a machine running an older lym, get none of this.

`~/.claude/settings.json` sets `effortLevel` per model under `modelSettings`, yet subagents with no `effort` ran at `xhigh` in `comment-cleanup`; whether a setting reaches a subagent's default is what `agent-skills-and-effort` checks.

## Wanted (operator, 2026-11-07)

- Heddle renders `.claude/settings.local.json` in every worktree, as it renders `CLAUDE.local.md`, so the operator can read what lym sets and every Claude session in the worktree gets it, one the operator starts by hand included.
  It holds the hub-wide keys: `attribution` empty, and the model and effort defaults from lym's config.
- Fabric triggers it through the worktree seam, on a worktree it adds and on the prime at `fabric clone`, which `clone-prime-heddle-unrendered` makes work.
- Heddle is the one place every per-worktree file lym seeds is declared, so a new one is a new heddle target.
- Claude Code writes the permissions the operator approves into `settings.local.json`, so heddle merges its own keys into the file and keeps every other key; it never overwrites the file whole.
  The file stays untracked, through the warp exclude list as `CLAUDE.local.md` does: fabric's `warp_exclude` key (`internal/fabricengine/template.yaml`, today `CLAUDE.local.md AGENTS.override.md .heddle`) gains `.claude/settings.local.json`, and fabric writes the list into the warp repository's `.git/info/exclude` before the seam fires, so heddle writes only after the line is in place.
  A hub whose `fabric.yaml` sets its own `warp_exclude` gets the new path through the config reconcile.
  Check whether heddle's targets should declare the paths they write, with fabric reading them from heddle, so a new target needs no second edit in fabric's config.
- What differs per session or role stays in the `--settings` document: the hooks with a run's paths, the session scope and creel's hooks.
