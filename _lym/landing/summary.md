# Report which session-scope settings work in a real Claude Code launch

Adds `docs/research/tokens/session-scope-launch-2026-11-06/report.md` and its line in `docs/research/README.md`.
No code changes.

The report records real launches in the sandbox hub on Claude Code 2.1.319, which the previous 2.1.318 binary reading could not do.
`prowler@scribe` off, `ly@loomyard` off over the project layer, `disableClaudeAiConnectors`, `disableBundledSkills`, `autoMemoryEnabled: false` and the `ListAgents` deny all work.
`skillOverrides` off does not hide a plugin's skills, and the `ly@loomyard` "on" case and the claude.ai skill override were not isolated from other settings.

The scope cuts a spawned role's start load from 38,463 to 37,330 tokens at its first request, and from 53,313 to 49,611 after a two-skill preload.
The orch's fresh-session start falls from 36,269 to 34,383.
That is far below the 31 percent projected, because scoped sessions load a `Workflow` tool schema about 5,800 tokens larger than unscoped ones; its cause is not established.

## Design decisions

- The role baseline came from a second launch with a local `session_scope.enabled=false` override, removed afterwards, so both rows share the Claude Code version.
- The sandbox orch's strand predated the scope, so its first cycle is the unscoped row; the scoped row needed a stop, a start and a second cycle.
- Two bug notes filed on the board: `plugin-skilloverrides-ignored` and `scope-enlarges-workflow-schema`.
- Figures are not compared with the startup-context report's 34,252 and 45,865, which measured other session shapes on 2.1.318.
