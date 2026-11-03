# codexengine: load skills and install every shell guard row on Codex

A Codex run can now load the skills a launch spec names, and it gets the same shell guard rows a Claude run gets.
Until now `codexengine` refused a launch that asked for the publish guard and installed no guard at all, so a Codex agent could run `python`, `sed -i` or a headless `claude` unchallenged.

Closes nothing on the issue tracker.

## What changes

- **Shell guards.**
  `codexengine` renders each installed `shellguard` row as one Bash `PreToolUse` hook.
  The hook greps the payload for the row's patterns and prints Codex's deny decision when all of them match; otherwise it prints the empty hook output.
  The prompt file now opens with the deny notice announcing each installed row.
- **Which rows install.**
  A row installs by its switch: always for `SwitchNone`, by the shuttle config's python switch for `SwitchDenyPython`, and by the launch spec's `DenyPublish` for `SwitchDenyPublish`.
  `validateSpec` no longer refuses `DenyPublish`.
  The refuse-raw list comes from the hub's gate config, loaded in `Prepare`; an absent config means the hub refuses nothing, and any other load failure refuses the launch.
- **Skills.**
  A spec that names skills launches with a one-line load turn that asks the model to read each skill's `SKILL.md` by absolute path and follow it, and `Prepare` returns the prompt pointer as `Launch.PromptLine` for shuttle to send afterwards.
  `Codex` implements `shuttleengine.SkillLoader`: `ClassifySkillLoad` reads the rollout the turn end names and reports each skill as loaded, missing or unknown.
- **Engine-specific texts in `shellguard`.**
  `Row` gains `EngineTexts`, with `SteerFor` and `NoticeFor`, so a row can word its steer for an engine whose tools differ.
  The python, shell-edit and headless-claude rows carry Codex texts: Codex has no Read, Edit, Write or Agent tool, so those steers point at `apply_patch` and at Codex's own subagents.
  `checkTexts` validates the engine texts as well as the plain ones.
- **Hook settings.**
  All guard entries share one `hooks.PreToolUse` setting, since a second `-c` of the same key replaces the first.
  `docs/overview.md` and the package docs are updated to match.

## Design decisions

- A skill is a file the model reads rather than a Codex skill the engine discovers: discovery outside a repository is untested, and a typed `$` opens a popup that takes the next Enter.
- `ClassifySkillLoad` recognises the load message by its fixed text, not by the paths it names, so a skill file that appeared after launch does not hide the turn.
  A skill counts as loaded when a tool call after that message names its `skills/<name>/SKILL.md` tail.
  The report is unverified when the rollout cannot be read or holds no load message.
- `installedGuards` panics on a row hanging on a switch the engine does not map, so a new switch cannot leave a row silently uninstalled.
- Skill roots come from the `codex.yaml` that `Prepare` already loaded, so one load serves the whole launch.

## To check by hand

- Run a real Codex launch with a spec that names skills, and confirm the model reads each `SKILL.md` and `ClassifySkillLoad` reports them loaded.
- Run a real Codex launch and try `python`, `sed -i`, a headless `claude` and a refuse-raw command, and confirm Codex honours each deny.
  The deny output shape follows what an earlier spike saw Codex honour; no test runs Codex itself.
- Deny output and Bash matcher: confirm `denyDecision` and `toolMatcherBash` in `internal/shuttleengine/codexengine/guards.go` stay pinned by a test against the literal output and matcher.
- Overview consistency: confirm `docs/overview.md` no longer says Codex has no skills or guards.
- Gate config load: confirm a `Prepare` test covers the gate config's load, both the absent and the failing case, and the python and publish switches through `Prepare`.
