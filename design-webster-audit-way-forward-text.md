# Webster run-exit audit: way-forward text ends mid-sentence and repeats paths

`webster-audit-way-forward-text` · Note · bug, webster

Imported from [issue #362](https://github.com/Knatte18/loomyard/issues/362).

When the Webster run-exit audit halts a run as `blocked` on forged contract files, the way-forward text it prints is garbled.

The halt note reads:

> way forward: reset the branch to the run's start commit with git and run "lyx webster run --fresh" and re-step the Webster row (lyx webster run), since nothing the run recorded can check `_lyx/webster/summary.md`, `_lyx/webster/outcome.yaml`

The final clause, "since nothing the run recorded can check <paths>", names no consequence and reads as truncated.
The same two paths are also printed twice: once in each finding and again as "suspect paths".
The way forward also tells the operator to run `lyx webster run --fresh` and then `lyx webster run`, without saying whether the second command is a separate step or a restatement of the first.

Expected: a complete sentence stating why the forged files cannot be trusted, and an unambiguous ordered list of commands.

Provenance: task `board-model`, note `loom-halt.md`.
