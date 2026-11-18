# Leftover strands and shuttle runs halt a gate re-prompt and a re-step

`leftover-strands-and-runs` · Task · bug, gate, shuttle, reed, shed

Depends on: [`orch-ignores-deny-python`](design-orch-ignores-deny-python.md), [`shuttle-start-idle-and-prompt`](design-shuttle-start-idle-and-prompt.md)

Two bugs where a leftover strand or shuttle run stops a run, bundled because both sit in shuttle run tracking and the step that meets it.

## Part: a gate re-prompt finds its strand untracked

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


## Part: a re-step halts on the shuttle run left by the interrupted stop

Imported from [issue #138](https://github.com/Knatte18/loomyard/issues/138).

### What happened

A `lym shed step lym-retro --until-stop` run halted `failed` at `Code-Burler-Review` with anomaly `interrupted-stop`: `loop-exited: loop ... ended without an envelope`, with no process marking the loop's exit.
The way forward in the status error was to read `loop.trace_copy` and `loop.stderr_path`, apply the interrupted rule, and re-run `lym shed step lym-retro --until-stop`.

The re-run halted the same way, again at `Code-Burler-Review`, with an identical 11-entry history and a new trace id.
The second halt's loop log carries this line:

`shuttle: a run reads running but its waiting process is gone ... strand=ly:lym-retro:code-burler-review pid=<pid>`

The pid in that warning equals the pid in the first halt's trace file name, so the run it names was left over from the first interrupted step.
The re-run found that run still reading `running`, and the loop ended without an envelope.

### Why it is abnormal

Following the way forward lym itself printed did not move the run forward: the retry tripped on the leftover from the interruption it was meant to recover from.
The history did not change between the two halts (the last entries stay `Code-Bouncer / stuck`), so the second attempt made no progress.
The note also says `Code-Burler-Review` was in flight although the last history entry is `Code-Bouncer`.

### What to do

- On a re-step after an `interrupted-stop`, reconcile a shuttle run that reads `running` whose waiting process is gone (mark it interrupted or abandoned) before starting the step, so the retry does not end on it.
- Check that the interrupted-step record names the step that was in flight, consistent with the history.
- The first loop exit left no exit marker; whether that was an actual kill or a machine stop could not be told from the notes.
  If a clean stop can look like this, record the cause where the loop exits.

Provenance: task slug `lym-retro`; notes `loom-halt.md`, `loom-halt-2.md`.

