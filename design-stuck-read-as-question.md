# A Webster stuck outcome is reported as the master asking a question

`stuck-read-as-question` · Note · bug, loom, shuttle

Seen 2026-10-07 in card-batching: the batten notice and loom status said `webster master is asking a question`, but the master had written `_lyx/webster/outcome.yaml` with `outcome: stuck` plus `summary.md` and ended its turn. The stop must be classified from the outcome file, not as a question; see `session-state` for the wider model.

## From issue #429

Imported from [issue #429](https://github.com/Knatte18/loomyard/issues/429).

**What happened**

During the Webster row, Master delegated a batch to a fork that ran a long after-state measurement in a background shell.
Master's turn ended with only interim fork output, because it was holding `record-batch` until the fork's final completion notification.
lyx's wait ran past `background_shell_wait_min` (10 minutes), stopped waiting and counted Master's turn end.
It did not stop the shell.
The run's outcome became `error: webster: master asked a question instead of finishing`, and the loom halted `blocked` with anomaly `escalation-to-human`.

Friction notes reported "Master's wait counted a turn end past a background shell" twice, in two different Master sessions.
The driver re-stepped the Webster row after a binary change (`last_step.binary_changed`) and Master blocked the same way a second time, still waiting on the same fork's measurement.
The halt recurred, so one re-step does not clear it.

**Why this is abnormal**

An interim "still waiting on a background job" turn end is not a question to a human, and halting the loom as `escalation-to-human` for it needs an operator for no reason.
The wait gives up on the shell after a fixed time, but the shell and the Master strand stay alive until the next `lyx webster run` reclaims them, so the work being waited on is killed or orphaned on the very re-step meant to recover.
The log line "stopping a leftover live strand before respawning it; an in-flight batch run restarts from state.json rather than resuming" shows the re-step discards the pending wait.

**What to do**

- Do not classify a Master turn end that is waiting on a live fork or background shell as a question or an `error`.
- Either keep waiting while the fork's background job is live, or resume Master when the job completes, instead of halting.
- If giving up after `background_shell_wait_min` stays, stop the shell and surface a distinct anomaly, not `escalation-to-human`.

Provenance: task `test-tiers`; notes `webster-background-shell.md`, `webster-background-shell-2.md`, `loom-halt.md`, `loom-halt-2.md`, `driver-restep-20261007-webster-blocked.md`.
