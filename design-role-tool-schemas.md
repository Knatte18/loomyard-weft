# Tool schemas are three quarters of a spawned role's start; drop the tools a role never calls

`role-tool-schemas` · Note · enhancement, shuttle, claude

## Seen

`docs/research/tokens/session-scope-fix-launch-2026-11-06/report.md`: a spawned role's first request is 29,087 tokens with the session scope on, and its tool schemas alone are 21,947 of them, about three quarters.
The role's tools are `Agent`, `Artifact`, `Bash`, `Edit`, `Read`, `ReportFindings`, `ScheduleWakeup`, `SendFeedback`, `Skill`, `ToolSearch`, `Write` and `advisor`.
Several are of no use to a darn writer or a reviewer, such as `Artifact`, `ScheduleWakeup`, `SendFeedback` and `advisor`.
The orch's tool schemas are 9,927 tokens; why a role's are more than twice that is not known.
The operator finds about 30k tokens at start too much (2026-11-07).

## Idea

Measure each tool's schema size in a role's request, then drop from each role the tools it never calls, through the session scope.
Check first which way of dropping a tool removes its schema from the request, `--disallowedTools` on the launch line or `permissions.deny` in the settings, since a deny rule that only refuses the call saves nothing.
