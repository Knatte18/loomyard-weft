## Goal

```markdown
## Seen (orch, 2026-11-01)

Dobby sent `[dobby notice] gate-fewer-binaries-trial: running at Darn-Write; no agent activity readable for 1h14m30s; since 2026-11-01T11:06:21Z; history 4` while the run was healthy.
The darn agent had made steady tool calls from 11:06 to 12:20:37 UTC, its longest gap a blocking gate call of about 11 minutes, and ended normally at 12:20:44.
At 12:20:44.6 shuttle set the session's `run.json` outcome to `done`; the producer's post-session commit followed at 12:20:45 and the row's history entry at 12:20:58.
A dobby probe at about 12:20:51 fell in that gap.

## Cause

- `shuttleengine.ReadAgentActivity` (`internal/shuttleengine/activity.go`) keeps only shuttle runs whose outcome is `running`, so in the gap it returns none.
- With no run, `judgeAgents` (`internal/dobbyengine/runnotice.go`) measures idle time from the later of the episode's `since` and the child's newest history entry, both of which here were the row's start (11:06:21), and words the notice "no agent activity readable".
- So for the few seconds between a session ending and its row's history write, a row that has run longer than the quiet window (30 minutes, `defaultNoticeQuiet`) reads as idle for its whole length.
  With a 30-second probe and a gap of about 13 seconds, that is roughly a 40% chance per such row; the same holds between session respawns inside one row.

## Idea

In the no-run fallback, the idle clock also counts the newest activity or end time of shuttle runs that have already ended under the task worktree, for example by `ReadAgentActivity` returning recently ended runs' last activity too, so a session that has just finished counts as fresh activity.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
