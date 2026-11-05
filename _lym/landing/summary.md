# Each Claude session gets only the plugins and skills its role needs

lym now writes a session scope into the settings file the Claude engine already passes each session with `--settings`.
That makes lym, not the user's settings or the hub repository's `.claude/settings.json`, the one that decides what a session loads.
The bug behind it: the user settings turn `ly@loomyard` off, and only loomyard's own checked-in settings turn it on, so the orch and board session in every other hub (scribe, quarry, lyt, cs) started without the `ly` skills and failed `Skill(ly:board)`.
The same settings also drop sources a session never uses, which `docs/research/tokens/startup-context-2026-11-05/report.md` puts at about 11,000 tokens per spawned role.

## What changes

- New `internal/shuttleengine/claudeengine/scope.go` works out the scope from `claude.yaml`'s new `session_scope` block, the skills the session preloads and the plugins Claude Code has installed:
  - Every plugin that `plugins/installed_plugins.json` lists, under `CLAUDE_CONFIG_DIR` or else `~/.claude`, goes into `enabledPlugins`. It is on only when the session preloads a skill under the plugin's name.
    So `ly@loomyard` is on for the orch and the board session in every hub and off for writers, reviewers and fixers. `scribe@scribe` is on wherever a role preloads scribe skills. `prowler@scribe` and plugins lym never names are off.
  - `disableClaudeAiConnectors` and `disableBundledSkills` are on in every session.
  - `skillOverrides` turns off each skill on the block's `skills_off` list unless the session preloads it. The template lists the `anthropic-skills` skills.
  - An autonomous run also gets every plugin skill except its preloaded ones turned off, so a Go writer keeps no Python or C# scribe skills. It also gets `autoMemoryEnabled: false` and `ListAgents` in `permissions.deny`.
- `Prepare` puts the scope into every run's settings. `OpenLaunch` puts it into an opened session's settings in a hub. With the scope off, both documents are unchanged byte for byte.
- `claude.yaml` gains `session_scope.enabled` (on) and `session_scope.skills_off`.
- The reed board button now preloads `scribe:prose` and `scribe:conversation` beside the two `ly` board skills, since the scope would otherwise leave the board session with no scribe plugin.
- Docs: the `claudeengine` package doc and the shuttle row of `docs/overview.md`.

## Design decisions

- Plugins follow the preloaded skills, not a role table. A role's stencil and `language.yaml` already list exactly the skills it needs, and an opened board session counts as the board session whatever name the operator gives it.
- Plugins come from Claude Code's install record rather than a list in lym. That way a plugin lym never names, such as the millhouse ones, is switched off too, and no plugin id is hard-coded. If that file can't be read, the error is logged and no plugin is named. The session then keeps its other settings' plugins, and connectors, bundled skills and `skills_off` are still turned off.
- Interactive sessions (the orch, an opened session, PR-Review, the sampler session) keep every skill of the plugins they have on, plus their auto-memory and `ListAgents`. The operator in the pane may call a skill no stencil preloads, such as `ly:distill`, and the orch keeps its memory as the record asks.
  This goes beyond the record's "scribe: only the preloaded skills" for the orch, whose other scribe skills stay listed.
- `ListAgents` is denied through `permissions.deny` in the settings, not `--disallowedTools` on the command line. One document then holds the whole scope, and the resume line keeps it without a second flag. Claude Code adds the deny to the rules of the other layers.
- `disableBundledSkills` is on for every session, the orch included. The record let the orch keep `update-config` only if it needed it, and nothing in the orch's role uses a bundled skill.
- An opened session outside a hub still writes no settings, so it gets no scope.
- Per the parent's decision added after Discussion, checking each setting in a real launch and measuring the start load move to note `session-scope-launch-trial`, a darn docs task released after this lands.

## Open risks

- No launch has tried the settings. From the Claude Code 2.1.318 binary, `skillOverrides` looks like it may apply to a plugin's skill only on its synced-skill path. If so, trimming scribe skills in autonomous runs has no effect until the note's trial settles it; the plugin switch, the connector and bundled-skill switches and memory do not depend on it.
- `skills_off` holds the `anthropic-skills` names as one account lists them today; an account that lists others needs its own list in `claude.yaml`.

## Tests

- `TestSessionScope_WrittenSettingsPerRole` checks the scope keys of the settings document for the orch, an opened session with no skills, a darn writer and a scope switched off.
- `TestReadInstalledPlugins` checks how the install record is read: each plugin with its skill directories, none for a missing file, an error for a malformed one.
- `TestSessionScope_ReachesWrittenSettings` checks that `Prepare` and `OpenLaunch` write the scope the shipped template turns on.
- The settings golden file is unchanged, since a zero scope adds no keys.
