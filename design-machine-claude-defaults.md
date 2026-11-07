# Heddle renders .claude/settings.local.json in every hub: no co-author, lym's plugins and tool deny list

`machine-claude-defaults` · Task · enhancement, claude, config, heddle, fabric · running · Drift-Baseline

Depends on: [`clone-prime-heddle-unrendered`](design-clone-prime-heddle-unrendered.md)

## Seen

The operator (2026-11-07) wants lym to set up, on the machine it runs on, the Claude Code settings lym needs to work as the operator wants:

1. No Claude co-author or session link in commits and pull requests; LoomYard has its own credit lines.
2. Model defaults that match lym's own, so no session or subagent runs at an effort lym did not choose, such as `xhigh`.

Today lym writes no `.claude/settings.json` into any worktree.
It writes a settings document per session and passes it with `--settings` (`internal/shuttleengine/claudeengine/settings.go`), and every such document already sets `attribution` to empty commit and pull-request texts (`780b1db4`), for run sessions and sessions opened in a hub alike.
A checked-in `.claude/settings.json` is the repository's own, which is why another machine's repositories differ.
A Claude session not started by lym, or lym sessions on a machine running an older lym, get none of this.

`~/.claude/settings.json` sets `effortLevel` per model under `modelSettings`, yet subagents with no `effort` ran at `xhigh` in `comment-cleanup`; whether a setting reaches a subagent's default is what `agent-skills-and-effort` checks.

## Two kinds of settings (operator, 2026-11-07)

- What lym needs: a repository initialized as a lym hub is developed only through lym, so lym's needs hold for every session in its worktrees.
  Everything in loomyard's checked-in `.claude/settings.json` but `mill@millhouse` is of this kind: `ly@loomyard` on, and the `permissions.deny` list of tools lym never uses.
  These ship with lym as heddle's shipped entries, and loomyard's checked-in `.claude/settings.json` is removed once they render.
  Every hub lym initializes gets the deny list, so its sessions save what this hub's do (operator, 2026-11-07): `docs/research/tokens/role-tool-schemas-2026-11-07/report.md` measured a role's first request at 13,449 tokens here against 29,087 in `lyt`, which lacks the list, and found no session on this machine that called `Artifact`, `ReportFindings`, `ScheduleWakeup`, `SendFeedback`, `advisor` or `Workflow`.
  `scribe@scribe` is on in this machine's `~/.claude/settings.json` and lym's roles preload its skills, so it ships too: the shipped entries set `enabledPlugins` `scribe@scribe` on and declare the marketplace under `extraKnownMarketplaces` (`scribe`, source GitHub `Knatte18/scribe`), as this machine's user settings do.
  Check in a launch that Claude Code offers to install a marketplace and plugin declared in `settings.local.json`, as it does for one declared in a project's settings.
  A machine that also declares `scribe` in its user settings with the same source gets the same result.
- `ly@loomyard` is only switched on, with no marketplace declared: its skills call the `lym` binary, so the plugin is installed together with that binary by lym's own install, never alone from a marketplace prompt (operator, 2026-11-07), and a declared source would also override this machine's `directory` marketplace that `update-plugins.sh` fills.
  A lym session whose preloaded skills are not installed already names the skill missing; lym also tells at hub start which plugin to install and how.
- What one machine needs: `mill@millhouse: false` is this machine's, not lym's.
  `~/.claude/settings.json` on this machine already sets it false, with `ly@loomyard` false too, so the repository line is redundant; machine settings stay in the user's own `~/.claude/settings.json`, and heddle carries no machine layer.

## Wanted (operator, 2026-11-07)

- Heddle renders `.claude/settings.local.json` in every worktree, as it renders `CLAUDE.local.md`, so the operator can read what lym sets and every Claude session in the worktree gets it, one the operator starts by hand included.
  It holds the hub-wide keys: `attribution` empty, the plugins and the deny list above.
  The model and effort defaults are left to `agent-skills-and-effort`, which builds the config they come from and may render them through this target.
- Fabric triggers it through the worktree seam, on a worktree it adds and on the prime at `fabric clone`, which `clone-prime-heddle-unrendered` makes work.
- Heddle is the one place every per-worktree file lym seeds is declared, so a new one is a new heddle target.
- Heddle merges into the file (operator, 2026-11-07): a key heddle sets overwrites the same key already there, and every other key stays, such as a permission Claude Code wrote on a "don't ask again".
- Heddle's entries hold text today, joined into markdown, so this target needs entries of a new kind: each entry's text is a JSON object, such as `{"permissions": {"deny": ["Workflow"]}}`.
  The render folds the entries in order, shipped first and the repository's after: objects merge key by key at every depth, a scalar from a later entry replaces an earlier one, and lists join without repeats, so two entries that each deny a tool both count.
  The folded object then merges into the file by the same rules, except that a heddle scalar overwrites the file's.
  A key heddle wrote at its last render that no entry sets any more is removed, so heddle records what it wrote, as in `.heddle`; a list item likewise.
  An entry whose text is no JSON object is refused when it is written.
  The file stays untracked, through the warp exclude list as `CLAUDE.local.md` does: fabric's `warp_exclude` key (`internal/fabricengine/template.yaml`, today `CLAUDE.local.md AGENTS.override.md .heddle`) gains `.claude/settings.local.json`, and fabric writes the list into the warp repository's `.git/info/exclude` before the seam fires, so heddle writes only after the line is in place.
  A hub whose `fabric.yaml` sets its own `warp_exclude` gets the new path through the config reconcile.
  Check whether heddle's targets should declare the paths they write, with fabric reading them from heddle, so a new target needs no second edit in fabric's config.
- What differs per session or role stays in the `--settings` document: the hooks with a run's paths, the session scope and creel's hooks.
