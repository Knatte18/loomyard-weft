# Gate re-prompt fails: shuttle strand not tracked by reed, Plan-Write halts blocked

`gate-reprompt-strand-untracked` · Note · bug, gate, shuttle, reed

Imported from [issue #143](https://github.com/Knatte18/loomyard/issues/143).

What happened: a `lym loom run` halted as blocked at Plan-Write with anomaly `escalation-to-human`, state `stuck`.
The reason reported by lym:

```
gate failed its first check and the session was never re-prompted, since its re-prompt could not be sent: shuttle: strand "<id>" is not tracked by reed — either its run completed and was cleaned up, or reed's strand table was reset under it (a reed remove/down, or a lost or rebuilt reed.json); check "lym reed status"
```

Why it is abnormal: the gate failed, and the re-prompt to the session could not be delivered because reed no longer tracked the strand.
Nothing in the task asked for a reed reset, so either the strand was cleaned up while the producer was still running, or reed's strand table was lost or rebuilt during the run.
The run stalled for about 23 minutes before it halted, and the findings file under `.lym/shed/<slug>/gate-findings/Plan-Write.md` was left for a human.
This run followed a separate failed run in the same worktree (see the stencil include issue), so a stale or reset reed table after the earlier halt is a plausible cause.

What to do: find why the strand was untracked at re-prompt time (cleanup racing the gate, or a reed.json reset after the earlier halt).
When the strand is gone, have the gate path relaunch or resume the producer instead of halting, or fail fast with this message rather than after a long wait.

Provenance: task slug `launch-manifest-parity`; notes `loom-halt-2.md`.
