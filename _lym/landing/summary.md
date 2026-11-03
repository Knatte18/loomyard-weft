# Name the skips and the scope change when a capture writes no file

A partial Drift scope whose `--skip` list covers every test the capture command runs leaves the harness nothing to profile.
The capture then exits 0, writes no file, and fails as `exit 0 but wrote no file to …`, a reason that names neither the skips nor a way out.
Both ways forward it offered fail: `lym drift capture` repeats the scope, and `lym drift defer` drops the regression the run was meant to have.
A tackler reading the stop could not find the cause and re-launched the same capture.

## What changes

- `RunCapture` marks a capture that exited 0 with no file as `wrote_no_file` in `capture.json` (new `CaptureRecord.WroteNoFile`).
- Under skipped tests, its reason now reads `… while scope partial skipped N tests, so every test the capture command ran may have been skipped`.
  N counts distinct skipped tests, and the wording is singular for one.
- A Drift-Baseline or Drift-Await Stuck over such a record no longer offers a plain `lym drift capture`.
  It names, in order:
  - `lym drift scope --scope full` (or `--scope partial` with a shorter `--skip` list), then `lym drift capture` and `lym loom resume`;
  - `lym drift scope --scope off`, then `lym loom resume`;
  - `lym drift defer`, then `lym loom resume`.
- Once HEAD has left the branch point no capture may run, so the Stuck drops the first way and offers only `--scope off` and `defer`.
- A capture that failed for another reason, or wrote nothing with no skipped tests, keeps its previous ways forward.

`doc.go` and the `CaptureRecord` field comment describe the new field and wording.

## Design decisions

- The scope-change ways are keyed on the recorded `wrote_no_file` flag plus a non-empty `Skip`, not on matching the reason text, so a reworded reason cannot silently disable them.
- The new field is a JSON addition through `internal/state`, so an older `capture.json` reads as `false` and keeps the plain ways forward.
- The reason cannot prove every test was skipped, since the harness's test set is not known to the engine, so it says "may have been skipped".

## Checks

Tests cover the reason wording (plural, singular, repeated skip), the Await and Baseline Stuck ways forward, and the unchanged cases.
No integration suite or run on the `cs` hub was done; the original failing scope (all six regression cases skipped) was not reproduced end to end.
