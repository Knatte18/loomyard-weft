# Trial the session-scope settings in a real launch and re-measure start load

`session-scope-launch-trial` · Note · research, claude, shuttle

## Seen

Task `ly-plugin-off-in-other-hubs` writes a session scope into every Claude session's `--settings` file (`internal/shuttleengine/claudeengine/scope.go`, `claude.yaml`'s `session_scope`).
The setting names come from the Claude Code 2.1.318 binary; no launch has tried them, and the darn writer could not, since a headless launch is refused in a run.

## Check in a launch

- `enabledPlugins` in the flag layer turns `ly@loomyard` on in a hub whose user settings turn it off, and `prowler@scribe` off.
- `skillOverrides` `off` hides a plugin skill (`scribe:python-testing`) and a claude.ai skill (`anthropic-skills:docx`) from the listing; the binary's resolver reads plugin skills' overrides only through the synced-skill path, so the plugin case may not work.
- `disableClaudeAiConnectors`, `disableBundledSkills` and `autoMemoryEnabled: false` drop their sources.
- `permissions.deny: ["ListAgents"]` removes the tool's schema, not only its calls.

## Measure

Re-run `docs/research/tokens/startup-context-2026-11-05/measure.sh` on one spawned role and one orch start after the change, against the report's 34,252 and 45,865 tokens.
