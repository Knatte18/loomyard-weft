# shed step re-halts on the shuttle run left running by the earlier interrupted-stop

`restep-leftover-shuttle-run` · Note · bug, shed, shuttle

Imported from [issue #138](https://github.com/Knatte18/loomyard/issues/138).

## What happened

A `lym shed step lym-retro --until-stop` run halted `failed` at `Code-Burler-Review` with anomaly `interrupted-stop`: `loop-exited: loop ... ended without an envelope`, with no process marking the loop's exit.
The way forward in the status error was to read `loop.trace_copy` and `loop.stderr_path`, apply the interrupted rule, and re-run `lym shed step lym-retro --until-stop`.

The re-run halted the same way, again at `Code-Burler-Review`, with an identical 11-entry history and a new trace id.
The second halt's loop log carries this line:

`shuttle: a run reads running but its waiting process is gone ... strand=ly:lym-retro:code-burler-review pid=<pid>`

The pid in that warning equals the pid in the first halt's trace file name, so the run it names was left over from the first interrupted step.
The re-run found that run still reading `running`, and the loop ended without an envelope.

## Why it is abnormal

Following the way forward lym itself printed did not move the run forward: the retry tripped on the leftover from the interruption it was meant to recover from.
The history did not change between the two halts (the last entries stay `Code-Bouncer / stuck`), so the second attempt made no progress.
The note also says `Code-Burler-Review` was in flight although the last history entry is `Code-Bouncer`.

## What to do

- On a re-step after an `interrupted-stop`, reconcile a shuttle run that reads `running` whose waiting process is gone (mark it interrupted or abandoned) before starting the step, so the retry does not end on it.
- Check that the interrupted-step record names the step that was in flight, consistent with the history.
- The first loop exit left no exit marker; whether that was an actual kill or a machine stop could not be told from the notes.
  If a clean stop can look like this, record the cause where the loop exits.

Provenance: task slug `lym-retro`; notes `loom-halt.md`, `loom-halt-2.md`.
