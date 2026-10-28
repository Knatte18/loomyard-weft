# Board entry `transient-halt-recovery`: A transient fork EAGAIN or died tmux session does not end a loom run hard

## Loom halts on transient fork/exec EAGAIN and records an opaque exit status 2 for the sibling halt (was note `loom-halt-on-transient-eagain`)

Imported from [issue #886](https://github.com/Knatte18/loomyard/issues/886).

Two loom halts were recorded within a second of each other (09:41:18Z and 09:41:19Z), both as `failed`, one in `Drift-Verify` and one in `Code-Bouncer`.

**What happened**

- The `Code-Bouncer` halt (anomaly `producer-hard-failure`) carries this lym error: `shedadapters: Code-Bouncer (bouncer): read the live subject: fabricengine: read task patch: diff the working tree against <sha>: fork/exec /usr/bin/git: resource temporarily unavailable`. That is EAGAIN from the OS while lym spawned `git`, a transient condition.
- The `Drift-Verify` halt (anomaly `interrupted-stop`) carries only `exited: exit status 2`. The note names no cause, and its history lists `Drift-Verify` as `skipped` even though the halt is attributed to it.

**Why it is abnormal**

- A transient process-spawn failure halts the whole run as `failed` with no retry, so a momentary resource spike costs the run.
- The two halts share a time window and most likely a root cause, but nothing in the notes links them.
- The `interrupted-stop` note gives no reason, so the operator must open the trace file to learn what exit status 2 means.
- The Drift-Verify history entry reads `skipped` while the halt names that producer as the failed one.

**What to do**

- Retry `fork/exec` EAGAIN in the fabricengine git wrapper with a short bounded backoff before surfacing it as a hard failure.
- Put the underlying cause (the failing command or its stderr tail) into `interrupted-stop` halt notes.
- Check whether lym itself spawns enough concurrent processes or git calls at the end of `Darn-Write` to exhaust the process limit.
- Reconcile the history state of a producer the halt blames with the halt note.

Provenance: task `review-setup-audit`; notes `loom-halt.md`, `loom-halt-2.md`.


#### From issue #891

Imported from [issue #891](https://github.com/Knatte18/loomyard/issues/891).

What happened: after the `Darn-Write` halt, a later loom entry halted `failed` at `Darn-Write` with `exited: exit status 2`, anomaly `interrupted-stop`, and an empty `trace_file:` field.
The history showed `Darn-Write / goto` as the last entry.
Afterwards the Go driver found the run `paused` at `Darn-Write` with no reason recorded, and wrote only a drive envelope under `_lym/shed/<slug>/drive-reports/`.

Why it is abnormal: an exit status 2 gives no cause and no trace file to follow, so the halt cannot be diagnosed.
A paused state with no reason leaves the operator and driver unable to tell whether the pause is deliberate or a leftover of the failed run.

What to do: make the `interrupted-stop` halt note always name the trace file and the exiting command, and record a reason whenever a run is left `paused`.

Provenance: task `hub-pull-verb`; notes `loom-halt-2.md`, `go-driver-paused.md`.

## Loom run fails hard when the producer's tmux session dies after a fork EAGAIN (was note `died-after-tmux-eagain-transient`)

Imported from [issue #890](https://github.com/Knatte18/loomyard/issues/890).

What happened: during `lym loom step` at the `Darn-Write` producer (seats engine), the trace shows `reed: tmux could not be started, retrying` with `fork/exec /usr/bin/tmux: resource temporarily unavailable`.
About ten seconds later shuttle reported `state=dead cause=process-gone`, the run finished with `outcome=die`, and the step was refused with `shedadapters: Darn-Write (seats): shuttle run outcome died` and an empty `transient` field.
The loom halted as `failed` with anomaly `producer-hard-failure`.

Why it is abnormal: the death coincides with a transient host resource shortage, yet the refusal carries no transient marker, so the run went to a terminal `failed` state instead of a retry or a resumable pause.
The halt note gives no hint that resource exhaustion preceded the death.

What to do: decide whether a `died` chair outcome that follows a tmux fork EAGAIN should be classed as transient, and either retry or surface the tmux error in the halt note.

Provenance: task `hub-pull-verb`; notes `loom-halt.md`.
