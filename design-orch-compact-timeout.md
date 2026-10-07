# Orch compact cycle always times out before reload

`orch-compact-timeout` · Note · bug, orch

## What happens

A compact cycle never sees its own compaction finish, so it always waits out the handoff timeout and aborts.
Seen 2026-10-07 on the hub orch:

- 12:05:39 the watcher enters `compacting` and types `/compact`.
- 12:06:41 the compaction boundary is written to the transcript.
- 12:15:41 the phase ends with `last_abort_reason: "compaction timed out"`.
- The next idle turn end finds the boundary through the auto-compaction path (`CompactedSince`) and only then sends the reload, about ten minutes late.

## Cause

`tickCompacting` (`internal/orchengine/watcher.go`) decides the compaction landed from the context reading of `st.ReadingTurnEnd`.
That field is written only in the idle phase (the `st.Phase == PhaseIdle && isTurnEnd(ev)` branch of the tick's event read), never while the phase is `compacting`.
So it stays on the turn end from before `/compact`, whose reading can never show `Compacted`, and the turn ends after the compaction are not read.

## Side effects

- The cycle counts as aborted: `cycle_count` does not move and `last_deferral` is set, which can push back the next automatic cycle.
- A `distill` request queued during the cycle is removed as stale (`orch: stale cycle request removed`) instead of being acted on.

## Fix direction

In `compacting`, take each turn end read after the phase entry as the reading candidate, or ask the transcript for a boundary after `PhaseEnteredAt` directly, as the auto path does.
A test drives a compact cycle through a turn end that follows the boundary and asserts the reload starts without the timeout.
