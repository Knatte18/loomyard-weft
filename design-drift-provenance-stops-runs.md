# Drift fails runs on capture noise such as _source.txt that is no part of the results

`drift-provenance-stops-runs` · Task · bug, drift, loom · done

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

- Lym writes no `_source.txt`: the Models hub's capture harness writes it into `{baseline_dir}` beside the results, the one directory drift compares.
  A capture command gets a second directory for metadata, such as a `{meta_dir}` placeholder, apart from `{baseline_dir}` (operator, 2026-11-07).
  Drift keeps it with the capture but never lists, compares or shows it to an agent as drift; the template and `drift init` say provenance goes there.
  The Models hub then moves its `_source.txt` there, a change in that hub's harness, which this run names in its landing.
  The baseline directory holds drift files only, the files that are compared; a metadata file such as the commit that made the baseline is kept, outside it (operator, 2026-11-07).
  The capture record already holds the HEAD a capture ran at; the landing's `## Regression check` names that commit, so a harness needs no provenance file of its own.
- The agent that decides what drift is acceptable tells metadata from results itself (operator, 2026-11-07): a capture's provenance, such as `_source.txt` with its commit SHA and capture time, timestamps, run ids and host names are metadata, not results.
  No hub lists them: `volatile_files` and `volatile_keys` are not the way, and no hub's `drift.yaml` needs them for metadata.
- Whatever agent sees the capture first, under `diff: lines` and `diff: command` alike, marks each metadata file and key, the run records the marks in its scope record, and no later listing, expectation, check or gate shows them to an agent or fails on them.
- The marks are named in the landing's `## Regression check`, so the operator sees what was left out.
- A gate failure that consists only of cells the writer cannot change ends the gate at once instead of re-prompting, and the parent gets one question, not one from the gate and one from the driver.
- Agents never call a noise cell "not equal": no stencil, listing or prompt shows one to them.
