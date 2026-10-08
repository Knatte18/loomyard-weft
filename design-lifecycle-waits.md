# Orch and batten: compact reload, one event-driven notice channel, a window per batten

`lifecycle-waits` · Task · bug, enhancement, orch, batten · awaiting · PR-Gate

The orch and batten side of a run's lifecycle: the orch reloads as soon as its compaction lands, it hears each state change once and on time, and it can see which batten runs it has in flight.

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

## One notice channel to the orch

Seen 2026-10-06: the orch gets two messages for every halt, and the batten one is often stale.

- The driver messages the orch at once with its stop report.
- Batten's Run-Shed row polls the child only every `poll_interval_s: 30` (`contracts/recipes/batten-recipe.yaml`), so its `[batten notice]` lags up to half a minute.
- The notice is typed into the orch pane and read only when the orch's current turn ends; by then the orch has often resumed the run, and the notice reports a state that no longer holds (twice on 2026-10-06: "paused" and "awaiting" for runs already running again; on 2026-10-07 most notices of the day were stale on arrival).

Direction:

- One channel per event: batten owns state changes (blocked, paused, awaiting PR-Gate, done) and its notice carries the drive report path; the driver messages the orch only with a question it cannot settle itself, never with "the run stopped".
  Parent-review requests and escalation briefs keep coming from the producer session.
- The notice is event-driven, not polled: batten wakes on the child's status-file change (or polls at a few seconds), since the orch relies on it as its only wake-up.
- Batten has no ticks (operator, 2026-10-07): the wait on the child is one step that blocks until the child's state changes, not a Run-Shed row that bounces `stuck` every 30 s.
  Batten's history records only the child's state changes, each with its timestamp; today every bounce adds a history entry (83 entries for `test-tiers` within an hour, all `Run-Shed → stuck`), which is noise, not history.
- The quiet notice measures the wrong thing: it fires when the child's status file is unchanged for 45 min, but the status file changes only at a step's end, so a live agent working a long step, or one already resumed, still trips it.
  On 2026-10-07 all five runs stalled on an API error; the quiet notices arrived 45 min later, long after the agents were resumed by hand, and said nothing about the cause.
  Quiet should be judged from the agent's own activity (transcript growth, turn ends) and should name an API-error turn end when that is what it sees.
- Each notice carries the time the state held and the child's history length, so a stale notice is recognisable without a status call.

## A window per batten

Seen 2026-10-03: the hub orch starts each batten detached with `setsid nohup lyx batten run <slug>`, so nothing on screen shows what the orch has running; Claude Code's footer shows only "1 shell" for a background helper, which says nothing about what it does.
Idea (operator): when the orch starts a batten, it lands in its own tmux window in the orch's session, so the window list names every run the orch has in flight.
`manifest/designs/reed-multi-window.md` sketches reed's own-window anchoring.

Related notes, not in this task: `orch-owns-the-loop` (lyx starts batten detached, a lyx watch, one land command) and `runs-overview` (one status line per run, the event log in its own window).
