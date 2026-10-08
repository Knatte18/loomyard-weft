# Loom re-step refused with producer error while a stopped review strand's run dir is younger than 3m

`restep-young-run-dir` · Note · bug, loom, burler

Imported from [issue #452](https://github.com/Knatte18/loomyard/issues/452).

## What happened

After a driver repair, the re-step of the Discussion burler segment failed with a non-transient `producer` error.
The trace line: `shed: step refused kind=producer transient="" error="... probe the burler-review half: shuttle: attach: 1 candidate run dir(s) ... cannot be confirmed dead or alive (untracked or with a cleared pane binding, younger than 3m0s) ..."`.
The run dir had been left by the review strand that the previous step itself stopped.
`lyx reed status` listed no such strand, while the burler-fix strand was still live, and no fabric mutation was half-finished.
The driver's only remedy was to wait until the run dir passed the 3m age and re-step, which cost a repair round (history_length 6).

## Why it is abnormal

The strand was stopped by lyx moments earlier, so its run dir is known dead, yet the burler probe treats it as unclassifiable and refuses the step.
A refusal that resolves itself with time should be a transient error, not a `producer` error that consumes a repair.

## What to do

- Have the stop path record or clear the stopped strand's pane binding, so the probe can classify its run dir as dead immediately.
- Alternatively, report the young-run-dir refusal as transient so the loom retries instead of halting for repair.

Provenance: task `plan-glyph-precision`; note `driver-repair-20261008-discussion-burler-2.md`.
