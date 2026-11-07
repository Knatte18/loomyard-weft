# Drift fails runs on capture noise such as _source.txt that is no part of the results

`drift-provenance-stops-runs` · Note · bug, drift

Imported from [issue #81](https://github.com/Knatte18/loomyard/issues/81); follows #71.

## Seen

Task run `support-for-dop-rheology-in` (hub Models, darn, `diff: lines`) failed the Darn-Write `drift` gate three times on the same two cells: keys `commit` and `captured_at` in case `_source.txt`, which the baseline harness writes on every capture.
No numerical cell moved.
The hub's `drift.yaml` had no `volatile_files`, because the fix for #71 made the ignore list opt-in.

- The writer could not fix the cells, so each re-prompt was a wasted round, and the writer had to message the parent.
- `lym drift amend` was refused while Darn-Write ran ("admitted only at its halt"), so the parent had to wait for the gate to use up its attempts before amending.
- After the halt, the driver also sent the parent the same question.
- A run keeps the volatile list it started with, so setting `volatile_files` during the run did not help that run.

The operator (2026-11-07): `_source.txt` is noise and is no part of the drift check; drift also reacts to much else that is irrelevant.

## Wanted

Drift checks only what the code's results are.
A capture's provenance and other cells that differ between two captures of the same code never fail a gate, never re-prompt a writer and never reach the parent, with no `drift.yaml` entry needed.

- Drift-Baseline captures twice on the branch point, the same code. Every file and every cell that differs between the two captures is noise: the run records it beside the volatile lists in its scope record, and no listing, expectation, check or gate sees it. A file present in one capture only counts as noise too.
- The recorded noise is named in the landing's `## Regression check`, so the operator sees what was left out.
- `volatile_files` and `volatile_keys` stay, for noise two captures in a row do not show.
- A gate failure that consists only of cells the writer cannot change ends the gate at once instead of re-prompting, and the parent gets one question, not one from the gate and one from the driver.
- Agents never call a noise cell "not equal": no stencil, listing or prompt shows one to them.
