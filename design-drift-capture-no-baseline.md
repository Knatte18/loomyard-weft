# lym drift capture: baseline capture exits 0 but writes no baseline file, halting loom at Drift-Await

`drift-capture-no-baseline` · Note · bug, drift, loom

Imported from [issue #41](https://github.com/Knatte18/loomyard/issues/41).

## What happened

The loom run halted blocked at the Drift-Await row.
Its reason: the baseline capture ended failed because it exited 0 but wrote no file to the drift baseline directory (`.lym/drift/baseline`).
The capture log lives under `_lym/drift/logs/`.
The halt recurred on the very next resume: Drift-Await went stuck, then goto, then stuck again with the identical reason, so the retry the way forward names did not recover it.
Only a later manual resume got the row to done.

## Why it is abnormal

A capture that exits 0 yet produces no baseline is either a capture that silently did nothing or a capture that wrote to the wrong place.
Either way the failure is only detected afterwards by Drift-Await, and the operator gets no cause beyond the log path.
The automatic re-capture on resume reproduced the same failure, so the halt looked deterministic until something outside the run changed.

## What to do

- Have `lym drift capture` fail with a non-zero exit and a stated cause when it produced no baseline file.
- Put the capture's stated cause into the Drift-Await halt reason, not only the log path.
- Investigate why the capture produced no file on this run and why the first re-capture reproduced it.

Provenance: task report-significant-digits; notes loom-halt.md, loom-halt-2.md.