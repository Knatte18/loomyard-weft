# Master's wait stops waiting on a still-running background shell at a turn end

`background-gate-shell-discarded` · Note · bug, webster, shuttle

Imported from [issue #505](https://github.com/Knatte18/loomyard/issues/505).

## What happened

Master's wait counted a turn end while a background shell was still running: an integration `lyx gate test --tags integration` run over several packages, piped through `grep` and `tail`.
The shell was reported only by the transcript, so the wait stopped waiting on it after `background_shell_wait_min` (10 minutes).
That turn end finished the run because Master's output files existed, and lyx did not stop the shell.
Shuttle then removed Master's strand, which ended the session and the shell with it.
The run's final outcome was `done`.

## Why it is abnormal

The integration test run was still in progress when the run finished, so its result was never seen by Master or the loom.
A run can report `done` with a gate tier unfinished, and the shell is killed silently rather than waited for or stopped explicitly.

## What to do

Decide whether a still-running background shell should hold the turn end past `background_shell_wait_min` when it is a gate run.
At minimum, make lyx stop the shell explicitly and record in the run that its result was discarded, so the outcome is not read as "tests passed".

Task: discussion-multi-llm. Notes: webster-background-shell.md

## Overlap

The shell-wait fault itself (#496) is in the scope of the running task `webster-replan-state`; this note adds the gate-run case, where a cut-off gate tier leaves the run reading as `done`.
Close it when that run lands if its fix covers a running gate shell.


## From issue #509

Imported from [issue #509](https://github.com/Knatte18/loomyard/issues/509).

Webster's wait stopped waiting on a background shell at Master's turn end.
The shells were `lyx gate test --tags integration` runs, still running when the wait counted the turn end.
The wait counts the turn end after `background_shell_wait_min` (10 minutes) for a shell only the transcript reports, and at once for one the Stop payload reports when Master's output files exist.
lyx did not stop the shell; shuttle removed Master's strand on finishing, which killed the session and the shell with it.
The same note was written for two separate shells in one run, and the run ended `stuck`.
In a later run the outcome was `done`, so the integration gate result Master was waiting for was never observed either way.

This is abnormal: the run can finish, or be held for the parent, while a verification Master started is still in flight and is then killed unobserved.

What to do: when a shell is still running at a turn end, either keep waiting until it exits or a hard limit, or stop it explicitly and record that its result was discarded, so the outcome does not depend on whether the output files already exist.

Provenance: task `driver-context`; notes `webster-background-shell.md`, `webster-background-shell-2.md`.


## From issue #517

Imported from [issue #517](https://github.com/Knatte18/loomyard/issues/517).

The `lyx webster` Master wait counted a Master turn end while several of Master's background shells were still running.
Per the note, the wait stops waiting on a shell at a turn end: after `background_shell_wait_min` (10 minutes) for a shell only the transcript reports, and at once for a shell the Stop payload reports when Master's output files exist.
The run then finished with outcome `done`, and lyx did not stop the shells; shuttle removing Master's strand ended the session and the shells with it.
The shells in question were `lyx gate test --tags integration ...` runs, so the run was declared done while integration test runs were unfinished and their results were never read.
Three such shells were recorded in one run.

This is abnormal because a run can finish as `done` with verification work in flight, and the result of that work is silently discarded.

What to do: when a turn end finds live background shells, either hold the run for the parent instead of finishing it, or stop the shells explicitly and record that their results were not consumed; at minimum surface this as a halt or warning rather than a friction note after the fact.

Provenance: task `webster-replan-state`, from note `webster-background-shell.md`.

## After `webster-replan-state` (5a8be0274)

5a8be0274 applies one shell-wait rule to both signals, keyed on the output files; it records every shell a run ends over in `Result.EndedShells`, lists them in webster's summary, and lets a stale shell hold the run until its own timeout instead of ten minutes.
#517 came from that run's own Master, which ran the old binary.
What remains: when the output files exist, a turn end still finishes the run as `done` over a live gate shell whose result nobody read; decide whether a live `lyx gate test` shell holds the run or turns its outcome into a warning.
