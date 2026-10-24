# Batten idle notice fires while a writer works in a fork

`idle-notice-misses-fork-activity` · Task · bug, batten · done

## Seen (orch, 2026-10-23)

Batten sent the orch an idle notice for `loom-tail-review` (creel #182) reading "no agent activity readable" for 1h15m, while the run was at Plan-Write and the Plan writer's fork was writing the plan.
The run was healthy: Plan-Write finished and Plan-Bouncer started at 14:05 local.
The writer's work was in its fork, so the likely cause is that the activity batten reads does not count a fork's transcript; that cause is not yet checked in the code.

## Idea

Batten's idle check also reads a session's fork transcripts too, under `subagents/` beside the session's own transcript, and counts their writes as activity, so a writer working in a fork raises no idle notice.
