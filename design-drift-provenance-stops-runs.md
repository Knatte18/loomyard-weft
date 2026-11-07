# drift: a capture's provenance file still stops runs by default and needs the parent to settle it

`drift-provenance-stops-runs` · Note · bug, drift

Imported from [issue #81](https://github.com/Knatte18/loomyard/issues/81).

## Seen

Task run `support-for-dop-rheology-in` (hub Models, darn, `diff: lines`) failed the Darn-Write `drift` gate three times on the same two cells: keys `commit` and `captured_at` in case `_source.txt`, which the baseline harness writes on every capture.
No numerical cell moved.
The hub's `drift.yaml` had no `volatile_files`, because the fix for #71 made the ignore list opt-in.

- The writer could not fix the cells, so each re-prompt was a wasted round, and the writer had to message the parent.
- `lym drift amend` was refused while Darn-Write ran ("admitted only at its halt"), so the parent had to wait for the gate to use up its attempts before amending.
- After the halt, the driver also sent the parent the same question.
- A run keeps the volatile list it started with, so setting `volatile_files` during the run did not help that run.

## Workaround

Set `lym config drift --set 'volatile_files=[_source.txt]'` from the prime for later runs.
For the run already going: wait for the Darn-Write halt, then `lym drift amend <slug> --entries` with keys `commit` and `captured_at` of case `_source.txt`, then `lym loom resume`.

## Expected

Provenance never reaches the parent:

- Drift detects a capture file whose only changes are commit and timestamp metadata, or ships `_source.txt`-style provenance as a default volatile entry.
- A gate failure that consists only of cells the writer cannot change ends the gate at once instead of re-prompting.
- `drift init` or `Drift-Baseline` warns when a capture holds a file that differs between two captures of the same code.
