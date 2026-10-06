# loom flags a clean producer handoff as a suspected driver crash

`false-crash-resume` · Note · bug, loom, shed

Imported from [issue #396](https://github.com/Knatte18/loomyard/issues/396).

## What happened

`lyx shed step` (the `step` verb) wrote a `loom-crash-resume` note ("loom resumed after a suspected driver crash at Webster-Bouncer") with `state: running`, `current_producer: Webster-Bouncer`.
The run history's last entry was `Webster / done`, recorded about six seconds before the step started.
The step then ran normally: a bouncer-seed shuttle run finished with `outcome=done`, and `shed: step done` reported `outcome=stuck next=Webster-Burler`.
Nothing in the trace shows a crash; the run was advancing from one producer to the next.

## Why it is abnormal

A crash-resume note is meant to mark a run that was last stopped without a clean handoff.
Here the previous producer completed and the next one had not yet started, so the note is most likely a false positive from the gap between two producers (the next row marked current, its run not yet begun).
The note also gives no reason for the "suspected" crash, so the operator cannot tell whether it is real.

## What to do

Check what `lyx shed step` / `lyx loom step` uses to decide that the previous driver crashed, and whether a run that is `running` with a freshly completed previous row should count.
Include the evidence (heartbeat, pid or lock state) in the note so a real crash and a false positive can be told apart.

Provenance: task `test-suite-measure`; notes: `loom-crash-resume.md`.


## From issue #392

Imported from [issue #392](https://github.com/Knatte18/loomyard/issues/392).

## What happened

`lyx shed step` wrote a `loom-crash-resume` friction note ("loom resumed after a suspected driver crash at Webster-Bouncer") for a run that had not crashed.

- The run's last history entry was `Webster / done`, recorded six seconds before the flagged step began.
- A `lyx board sync` ran in that gap, as it does after a finished producer.
- The flagged step itself ran normally: `shuttle: run started` then `run finished outcome=done`, then `shed: step done producer=Webster-Bouncer outcome=stuck state=running next=Webster-Burler`.
- The status was `state: running` with `current_producer: Webster-Bouncer`, which is the ordinary state between two steps.

## Why it is abnormal

A clean handoff between two steps was classified as "last stopped without a clean handoff".
The note sends the reader looking for a driver crash that did not occur.

Two smaller defects in the same note:

- It reports `history_entries: 14` but lists only five history lines, with no marker that the list is truncated.
- Its history ends at `Webster / done` and omits the later entries, so it cannot be used to tell whether the stop was clean.

## What to do

- Check the crash-detection condition in the `lyx shed step` / `lyx loom step` entry path: a run whose last producer finished `done` and whose next producer has not started should not count as crashed.
- State in the note when the history list is truncated, or list all entries.

## Provenance

Task slug `board-config-per-worktree`; note file `loom-crash-resume.md`.
