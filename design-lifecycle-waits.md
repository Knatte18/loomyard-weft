# Run lifecycle waits: orch compact reload, a step watchdog, a pair create that retries

`lifecycle-waits` · Task · bug, enhancement, orch, shed, fabric

Three places where a run or the orch waits far longer than it needs to, or hangs unnoticed.

## Orch compact cycle times out before reload

A compact cycle never sees its own compaction finish, so it always waits out the handoff timeout and aborts.
Seen 2026-10-07 on the hub orch:

- 12:05:39 the watcher enters `compacting` and types `/compact`.
- 12:06:41 the compaction boundary is written to the transcript.
- 12:15:41 the phase ends with `last_abort_reason: "compaction timed out"`.
- The next idle turn end finds the boundary through the auto-compaction path (`CompactedSince`) and only then sends the reload, about ten minutes late.

Cause: `tickCompacting` (`internal/orchengine/watcher.go`) decides the compaction landed from the context reading of `st.ReadingTurnEnd`.
That field is written only in the idle phase (the `st.Phase == PhaseIdle && isTurnEnd(ev)` branch of the tick's event read), never while the phase is `compacting`.
So it stays on the turn end from before `/compact`, whose reading can never show `Compacted`, and the turn ends after the compaction are not read.

Side effects:

- The cycle counts as aborted: `cycle_count` does not move and `last_deferral` is set, which can push back the next automatic cycle.
- A `distill` request queued during the cycle is removed as stale (`orch: stale cycle request removed`) instead of being acted on.

Fix direction: in `compacting`, take each turn end read after the phase entry as the reading candidate, or ask the transcript for a boundary after `PhaseEnteredAt` directly, as the auto path does.
A test drives a compact cycle through a turn end that follows the boundary and asserts the reload starts without the timeout.

## A step that never returns

From run-recovery PR-Gate (2026-10-07): run-recovery gave `verifytree` a 60-minute timeout and kills the process group on Unix.
Still open from #420: a step-level watchdog, so a `lyx shed step` that never returns is noticed without the hub reading panes; and on Windows the timeout kills only the shell, not the process tree it started.
