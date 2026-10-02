# Card 2 — spec-allow-agent-tool

**Edit:**
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/settings_test.go`

**Intent:**
Add `AllowAgentTool bool` to `Spec`, engine vocabulary like `ForkSubagents`: when true, the run installs no Agent `PreToolUse` deny and announces none, whatever shuttle's `claude_deny_agent_tool` key says.

In `settings.go`, `denyInstalls` takes the allowance and reports no Agent deny when it is set;
`buildSettings` and `buildDenyNotice` pass it through, so the hook and its notice sentence still come from one decision and cannot drift.
The fork-context `lyx webster` guard stays keyed on `forkSubagents` alone and is installed whenever `ForkSubagents` is set, allowance or not.
`Prepare` passes `spec.AllowAgentTool` to both builders.
Update the file doc comment and `denyInstalls`'s doc comment to name the per-run override.

Bound: the allowance lets every subagent type through the Agent tool for that one run only.
Only a caller that sets the field gets it; every other run keeps shuttle's config-driven deny, and no shuttle config key or `lyx shuttle run` flag reaches it.

Tests, each with `claude_deny_agent_tool` on: the allowance removes the Agent hook and the Agent notice sentence without `ForkSubagents`;
it removes them with `ForkSubagents` too, while the Bash webster guard and the `CLAUDE_CODE_FORK_SUBAGENT` wrapping stay;
and the AskUserQuestion deny and its notice sentence are unaffected in an autonomous run.
Extend the `internal/shuttleengine` package doc paragraph from card 1 to name `AllowAgentTool`.

**ImpactSummary:** Adds one Spec field and one parameter to the claudeengine deny decision; runs that leave it false produce byte-identical settings and notices.
