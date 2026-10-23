# Batten idle notice fires while a writer works in a fork

`idle-notice-misses-fork-activity` · Note · bug, batten

## Seen (orch, 2026-10-23)

Batten sent the orch an idle notice for `loom-tail-review` (creel #182) reading "no agent activity readable" for 1h15m, while the run was at Plan-Write and the Plan writer's fork was writing the plan.
The run was healthy: Plan-Write finished and Plan-Bouncer started at 14:05 local.
The activity batten reads did not count the fork's own transcript, so a writer working through a fork looked idle.

## Idea

Batten's idle check reads a session's fork transcripts too, under `subagents/` beside the session's own transcript, and counts their writes as activity, so a writer working in a fork raises no idle notice.
