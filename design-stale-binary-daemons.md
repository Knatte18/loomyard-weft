# Long-lived lyx processes restart on a new binary

`stale-binary-daemons` · Note · bug, reed, orch, loom

## Idea (operator, 2026-10-13)

Long-lived lyx processes keep running the binary they started with after a deploy replaces `lyx`, and nothing restarts them.
They should pick up the new binary on their own.

## Seen (2026-10-13)

- The hub's `lyx reed watchdog` (single instance per hub, spawned by `up`, `resume` and `attach` only when no daemon holds the lock) ran from 2026-10-08 on a deleted binary.
  It re-applied the layout of every session every 2 s and failed each time with an error only the old binary raises (`deferred anchor "own-window"`), and each attempt unzoomed and re-zoomed the strands' window (`internal/reedengine/reapply.go:130`, `zoom.go:57-76`): the zoom flicker in `reed-window-order`.
  It also filled its trace log to the size cap (`_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log`); the orch killed it and started a new one by hand.
- After the kill, other lyx processes still ran a deleted binary (`/proc/<pid>/exe` marked deleted): `lyx orch watch` (since 2026-10-10), a run's `lyx loom checklist` status pane (started before the `run-window-bugs` deploy, so it lacks the status-pane fix) and an operator's `lyx reed attach`.
- Shed already records a build identity per step and reports `binary_changed` (`internal/shedverbs/steprecord.go`), so a build identity to compare against exists.

## Scope to discuss

- Which processes restart themselves: the reed watchdog, status panes (`lyx loom checklist`, batten's overview), `lyx orch watch`, and any other long-lived verb.
- How: each daemon notices a changed binary (its own executable deleted or a new build identity) and re-execs itself with the same arguments at a safe point; or the deploy (`update-plugins.sh` and batten's Land deploy) signals or restarts every running daemon.
- A single-instance daemon hands its lock over across the re-exec, so no second instance starts in the gap.
- Processes that must not restart mid-work (an agent session, a step in flight) are only reported, for instance in `lyx reed status`.
