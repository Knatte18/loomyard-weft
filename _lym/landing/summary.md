# Drift keeps capture metadata such as _source.txt out of every check

A capture's provenance, such as a `_source.txt` holding the commit the capture ran at and its capture time, differs in every capture whatever the code does.
Drift compared it as a result, so a darn run failed its `drift` gate three times on cells no writer could change, and the parent got the same question twice.
After this change, drift compares only a capture's results, and no hub needs a `drift.yaml` entry for its metadata.

## What changes

- **A metadata directory for the capture.**
  A capture or compare command may hold the new `{meta_dir}` placeholder beside `{baseline_dir}`.
  It renders as a `.meta` sibling of the capture directory: `.lym/drift/baseline.meta` for the baseline, `<scratch>.meta` for a check's current capture.
  Drift creates it, moves it with the baseline to `baseline.N.meta` when a new capture backs the baseline up, and never lists, compares or shows it to an agent.
  The `drift.yaml` template says provenance goes there.
- **The expect agent marks metadata.**
  `expected.yaml` takes a top-level `metadata: {files: [...], keys: [...]}` under both diff modes.
  The expect stencil has the agent tell metadata from results itself: provenance, timestamps, run ids and host names.
  `ParseExpectation` checks each mark against the baseline, and refuses `keys` under `diff: command`.
  Drift-Expect records the marks in the scope record's new `metadata` field when it seals.
  `Check` drops the cells of every mark, from the scope record, the sealed file and any `lym drift amend --entries` amendment.
  Under `diff: command`, a judge's ruling on a marked case counts as `matches`, and a stored ruling on one stops nothing.
- **The `drift` gate stops at once on cells no code change can move.**
  Before it fails on unexpected cells, the gate marks a cell as unchangeable when either holds:
  - its old value names the baseline's capture commit and its new value the code HEAD;
  - a second current capture of the same code gives it another value.
  The writer's findings never name an unchangeable cell.
  When every unexpected cell is unchangeable, the gate returns a `Terminal` failure: the writer is not re-prompted, and the run's parent gets one stop.
  That stop names the cells and the way forward, `lym drift amend <slug> --entries <file>` with `metadata` marks.
  The gate now needs the `Head` seam, which `shedrecipe` requires for the `drift` entry.
- **The landing names what was left out.**
  The `## Regression check` section gains a `Baseline: captured at` line, from the oldest `Drift-Baseline` commit.
  It also gains a line naming the files and keys marked as metadata.
  A harness thus needs no provenance file of its own.

## Design decisions

- `volatile_files` and `volatile_keys` stay as hub keys for other noise, but their template text no longer suggests them for provenance.
  Retiring them would need a reinterpretation and a migration of recorded scope records, which the board entry does not ask for.
- The board entry says the template and `drift init` name the metadata directory.
  No `drift init` verb exists, so the `drift.yaml` template and the package doc carry that text.
- The second capture runs only when a failing gate has an unexpected cell that the commit rule leaves.
  A gate that passes, or one that fails on a capture fault, takes one capture as before.
- Drift-Verify keeps stopping on an unexpected cell, unchangeable or not.
  The parent's `lym drift amend --entries` with `metadata` marks clears such a cell.

## Follow-up outside this repository

- The Models hub's capture harness writes `_source.txt` into `{baseline_dir}`.
  It should take `{meta_dir}` in its `capture.command` and write `_source.txt` there.
  Until it does, the expect agent marks `_source.txt` as metadata in each run, and the gate's commit rule catches the `commit` cell.
