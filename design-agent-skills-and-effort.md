# Skills and effort for every role and subagent come from config

`agent-skills-and-effort` · Task · bug, shuttle, stencil, claude, gate, loom · draft

Depends on: [`claude-session-fixes`](design-claude-session-fixes.md), [`machine-claude-defaults`](design-machine-claude-defaults.md)

Three bugs bundled by the operator (2026-11-07): which skills and which effort an agent runs with is decided in lym's config, for every role lym starts and for every subagent such a role spawns.

## Seen

- Skills per role are Go lists: `darnSkills`, `roleSkills` (Webster), `burlerSkills`, `fixerSkills` and `conflictSkills`, each `scribe:prose`, `scribe:code-quality` and `scribe:testing`, joined with `skills` from `language.yaml` through `languageconfig.AppendSkills`.
  The gate fixer has no list of its own: `internal/loomcli/gatefixer.go` and `internal/gatecli/detachfixer.go` pass `AppendSkills(nil, ...)`, so it loads only the language skills, with no `scribe:prose` or `scribe:code-quality`, and edits Go files (seen in `creel-cross-hub-build`, `python-guard-gives-no-reason` and `drift-halts-bundle`).
- A subagent a role spawns through the Agent tool gets a skill only if its brief names one: in `comment-cleanup` none of the 502 subagents made a Skill call, and the comments the repository's earlier workers wrote missed the scribe rules for it.
- A subagent spawned with no `effort` ran at `xhigh`: in `comment-cleanup` Haiku 5.5 workers carry about 36,000 records with `"effort":"xhigh"`, though `~/.claude/settings.json` sets `effortLevel` medium for `claude-haiku-5-5` under `modelSettings` and the operator never sets xhigh.
  Lym passes `--effort` to a session it launches only when a model-spec names one, and passes nothing for a subagent.

## Wanted

- **Skills in config.** One config lists the skills each role lym starts loads, by role, the gate fixer included, in place of the Go lists; `language.yaml`'s `skills` stays the hub's language layer on top.
  A role missing from the config is a refusal at load, so no role starts with no list.
- **Subagents inherit.** A role that spawns subagents through the Agent tool tells each to load the skills from its own list that fit the subagent's work: a subagent that writes or reviews code or comments loads the prose, code-quality and language skills.
  The stencils and the briefs lym writes say so, and the way the parent learns its own list is one place, not restated per stencil.
- **Effort controlled.** Every session lym launches gets an explicit effort, from config by role and model where its model-spec names none.
  Every subagent a role spawns gets an explicit `effort` in its Agent call, from the same config, so no subagent falls to Claude Code's default.
  Check whether a Claude Code setting or environment variable pins a subagent's default effort, and set it as a backstop if one exists, through the `.claude/settings.local.json` heddle renders where it is a setting.