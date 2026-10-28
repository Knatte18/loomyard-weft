# Loom run fails hard when the producer's tmux session dies after a fork EAGAIN

`died-after-tmux-eagain-transient` · Note · bug, shuttle, reed, shed

Imported from [issue #890](https://github.com/Knatte18/loomyard/issues/890).

What happened: during `lym loom step` at the `Darn-Write` producer (seats engine), the trace shows `reed: tmux could not be started, retrying` with `fork/exec /usr/bin/tmux: resource temporarily unavailable`.
About ten seconds later shuttle reported `state=dead cause=process-gone`, the run finished with `outcome=die`, and the step was refused with `shedadapters: Darn-Write (seats): shuttle run outcome died` and an empty `transient` field.
The loom halted as `failed` with anomaly `producer-hard-failure`.

Why it is abnormal: the death coincides with a transient host resource shortage, yet the refusal carries no transient marker, so the run went to a terminal `failed` state instead of a retry or a resumable pause.
The halt note gives no hint that resource exhaustion preceded the death.

What to do: decide whether a `died` chair outcome that follows a tmux fork EAGAIN should be classed as transient, and either retry or surface the tmux error in the halt note.

Provenance: task `hub-pull-verb`; notes `loom-halt.md`.
