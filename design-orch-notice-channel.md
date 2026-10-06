# One prompt notice channel to the orch: batten, event-driven, timestamped

`orch-notice-channel` · Note · enhancement, batten, orch

Seen 2026-10-06: the orch gets two messages for every halt, and the batten one is often stale.

- The driver messages the orch at once with its stop report.
- Batten's Run-Shed row polls the child only every `poll_interval_s: 30` (`contracts/recipes/batten-recipe.yaml`), so its `[batten notice]` lags up to half a minute.
- The notice is typed into the orch pane and read only when the orch's current turn ends; by then the orch has often resumed the run, and the notice reports a state that no longer holds (twice on 2026-10-06: "paused" and "awaiting" for runs already running again).

## Direction

- One channel per event: batten owns state changes (blocked, paused, awaiting PR-Gate, done) and its notice carries the drive report path; the driver messages the orch only with a question it cannot settle itself, never with "the run stopped".
  Parent-review requests and escalation briefs keep coming from the producer session.
- The notice is event-driven, not polled: batten wakes on the child's status-file change (or polls at a few seconds), since the orch relies on it as its only wake-up.
- Each notice carries the time the state held and the child's history length, so a stale notice is recognisable without a status call.

Related notes: `orch-owns-the-loop` (detached batten start, a lyx watch) and `runs-overview` (notices shortened, the event log in its own window).
