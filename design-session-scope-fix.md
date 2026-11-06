# Session scope: drop the larger Workflow schema and hide plugin skills that are set off

`session-scope-fix` · Task · bug, claude, shuttle · running · Code-Bouncer

Two faults in the session scope, found by the launch trial `docs/research/tokens/session-scope-launch-2026-11-06/report.md` on Claude Code 2.1.319.
The scope lives in `internal/shuttleengine/claudeengine/scope.go` and `claude.yaml`'s `session_scope`.

## 1. Scoped sessions load a Workflow tool schema about 5,800 tokens larger

The `Workflow` tool schema is 22,339 characters with the scope on and 5,384 without it, in a spawned role and in the orch alike; the extra is its workflow authoring reference.
At the first request this gives back about 5,800 of the roughly 7,500 tokens the scope saves.

Find which setting triggers it, then remove the cost.
Leads, none confirmed:

- `disableBundledSkills`: Claude Code ships a bundled `workflow-authoring` skill, and with it gone the reference may move into the tool's schema.
  The setting's own description in the binary says it removes the skills and workflows that ship with Claude Code.
  The report's lead was `disableClaudeAiConnectors`, set in the same launch, so the two are not isolated.
- The binary has a `disableWorkflows` setting, which may drop the tool whole.
- `permissions.deny` dropped the schema of `ListAgents` and may do the same for `Workflow`.

First check that no session lym launches uses `Workflow`, and keep it where one does.

## 2. `skillOverrides` off does not hide a plugin's skills

A spawned role with twelve `scribe` skills set `off` still lists all fourteen.
The binary reads a plugin skill's overrides only through the synced-skill path.
The operator's aim stands: skills only where they are needed.

Find a setting that works for a plugin's skills and use it.
The binary speaks of a session skills allowlist, which is one lead.
If none works, say so in the change description, drop the `skillOverrides` entries that do nothing, and state what stays listed.

## Check

Each fix is proved in a real launch: `lym shuttle open` in the sandbox hub `lyt`, as the trial did, at most four launches, with no headless launch.
State the first-request tokens of a spawned role, scope on, against the report's 37,330.
A session that needs a skill it preloads still has it.
