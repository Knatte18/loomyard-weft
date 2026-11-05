# Scope plugins, skills and connectors per session in every hub

Every session lym launches now gets an explicit scope in the settings file lym already passes with `--settings`.
The scope decides which plugins, skills, connectors, memory and tools the session loads, over the user's settings and the repository's `.claude/settings.json`.
No file is added to any hub's repository.

Why: in the scribe hub the orch failed `Skill(ly:board)` with "Unknown skill", because the user settings turn `ly@loomyard` off and only loomyard's own checked-in settings turn it on.
Meanwhile every session in every hub loaded plugins, skills and connectors it never used, which `docs/research/tokens/startup-context-2026-11-05/report.md` puts at about 11,000 tokens per spawned role and 7,900 per orch start.

## What changes

- An installed plugin is on exactly when the session preloads a skill of that plugin; otherwise it is off.
  The ly plugin is therefore on for the orch and the board session in every hub, and off for sessions that preload none of its skills.
- A plugin that is on keeps only the skills the session preloads, except the ly plugin, which keeps all its skills because the operator types them as slash commands.
- claude.ai connectors (`disableClaudeAiConnectors`) and bundled skills (`disableBundledSkills`) are off in every session.
- The skills named in `skills_off` are off in every session that does not preload them.
  The template lists the `anthropic-skills` skills.
- Only the orch keeps auto-memory and the `ListAgents` tool.
  Every other session, interactive or not, gets `autoMemoryEnabled: false` and `ListAgents` under `permissions.deny`.
- The scope applies to run sessions (`buildSettings`) and to sessions opened in a hub (`buildOpenSettings`), through the new `scope.go` in `internal/shuttleengine/claudeengine`.
- A new `session_scope` block in `claude.yaml` (`enabled`, `skills_off`) switches the scope off or edits the skill list.
  Disabled, the settings carry none of the scope keys.
- The board button in both reed templates now preloads `scribe:prose` and `scribe:conversation`, so the board session keeps those rules once plugins follow preloads.

## Design decisions

- Installed plugins and their skills are read from Claude Code's config directory (`$CLAUDE_CONFIG_DIR`, else `~/.claude`), file `plugins/installed_plugins.json`.
  An unreadable directory or plugins file is logged and treated as no plugin installed; connectors and bundled skills are still turned off.
- The orch is recognised by strand role, via `agentname.MatchesRole`.
- `ListAgents` is denied through `permissions.deny` in the written settings rather than `--disallowedTools` on the launch line, so one settings document carries the scope and the resume line needs no second flag.
- A session opened outside a hub writes no settings and so gets no scope, as before.

## Not done here

Checking in a live launch that each lever works, and measuring the start load of a spawned role and of the orch before and after, is left to the follow-up docs task `session-scope-launch-trial`.
Nothing in this change was run against a real Claude launch.

## To check by hand

- The board session preloading `scribe:prose` and `scribe:conversation` (reed templates, `internal/reedengine/config_test.go`) was not in the original task text.
- Interactive sessions other than the orch (PR review, the sampler, the board session) lose auto-memory and `ListAgents`, and non-ly plugins keep only preloaded skills; the original task text exempted interactive sessions.
- `ListAgents` is denied in `permissions.deny` and not on the launch line.
- A session opened outside a hub gets no scope.
