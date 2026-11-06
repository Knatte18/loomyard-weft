# Trial the session-scope settings in a real launch and re-measure start load

`session-scope-launch-trial` · Task · research, claude, shuttle · running · Drift-Baseline

## Seen

Task `ly-plugin-off-in-other-hubs` writes a session scope into every Claude session's `--settings` file (`internal/shuttleengine/claudeengine/scope.go`, `claude.yaml`'s `session_scope`).
The setting names come from the Claude Code 2.1.318 binary; no launch has tried them, and the darn writer could not, since a headless launch is refused in a run.

## How to run

Launch real sessions through `lym shuttle open` or `lym shuttle run` in the sandbox hub `lyt` (`/home/knatte/Code/lym-test-LYMHUB`), so they run in a pane and no headless launch is needed.
Read what a session has from its pane, its transcript and its settings file.
Cap: one spawned role and one orch start, each launched at most twice.
A Claude Code version newer than 2.1.318 on this machine is the one to try; name the version in the report.

## Check in a launch

- `enabledPlugins` in the flag layer turns `ly@loomyard` on in a hub whose user settings turn it off, and `prowler@scribe` off.
- `skillOverrides` `off` hides a plugin skill (`scribe:python-testing`) and a claude.ai skill (`anthropic-skills:docx`) from the listing; the binary's resolver reads plugin skills' overrides only through the synced-skill path, so the plugin case may not work.
- `disableClaudeAiConnectors`, `disableBundledSkills` and `autoMemoryEnabled: false` drop their sources.
- `permissions.deny: ["ListAgents"]` removes the tool's schema, not only its calls.

## Measure

Re-run `docs/research/tokens/startup-context-2026-11-05/measure.sh` on one spawned role and one orch start after the change, against the report's 34,252 and 45,865 tokens.

## Output

`docs/research/tokens/session-scope-launch-2026-11-06/report.md`, with a line in `docs/research/README.md`.
The report opens with which settings work, which do not, and the start load before and after.
File each setting that does not do what it should as a bug note.
