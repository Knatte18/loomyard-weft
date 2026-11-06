# Session scope: drop the Workflow tool and dead plugin skill overrides

The session scope in `internal/shuttleengine/claudeengine/scope.go` had two faults, found by the launch trial `docs/research/tokens/session-scope-launch-2026-11-06/report.md` on Claude Code 2.1.319.

## Workflow schema

With the scope on, a session loaded a `Workflow` tool schema of 22,339 characters instead of 5,384.
The extra is the workflow authoring reference.
`disableBundledSkills` removes the bundled `workflow-authoring` skill, and the reference then moves into the tool's schema.
That gave back about 5,800 of the roughly 7,500 tokens the scope saves at the first request.

The scope now also writes `disableWorkflows: true`, which drops the tool.
No session lym launches calls `Workflow`, so nothing is lost.

## Plugin skill overrides

`skillOverrides` set to `off` does not hide a plugin's skills.
Claude Code 2.1.319 reads those overrides for a plugin's skill only through its claude.ai skill sync.
A `Skill(...)` deny rule refuses the call but leaves the listing, and the session skills allowlist reaches only an SDK session.
No setting hides one skill of a plugin that is on.

So the scope no longer builds per-skill overrides for a plugin's skills.
A plugin that is on lists all its skills; a plugin the session does not preload from is still off whole through `enabledPlugins`.
A plugin's skill named in `skills_off` stays listed too; the override is only effective for non-plugin skills such as `anthropic-skills:*`.
The scope code no longer reads each plugin's `skills` directory, and `readInstalledPlugins` returns only the sorted plugin ids.
The `ly` plugin special case is gone, since every plugin that is on now keeps all its skills.

## Design notes

- `template.yaml`, `doc.go` and `docs/overview.md` state the new behavior and the Claude Code 2.1.319 bound.
- `scope_test.go` pins `disableWorkflows` in each role's settings and drops the tests of the skills-directory reading.

## To check by hand

The decision record asks that each fix be proved in a real `lym shuttle open` launch in the sandbox hub `lyt`, with the first-request tokens of a spawned role, scope on, stated against the report's 37,330.
The diff carries no such measurement, and the unit tests pin only the written settings.
Run that launch before relying on the token saving.

Closes the two faults of the board task `session-scope-fix`; no GitHub issue is linked.
