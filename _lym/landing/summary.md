# An empty capture under skipped tests names the skips and a scope change

A partial Drift scope whose `--skip` list covers every test the capture command runs ends in a capture that exits 0 and writes no file.
The failure reason said only `exit 0 but wrote no file to <dir>`, and the Stuck offered `lym drift capture` with `lym loom resume`, which repeats the same scope and fails the same way.
A tackler reading that stop could not find the cause and re-launched blind.

## What changes

- `RunCapture` marks such a record with the new `wrote_no_file` key.
  Under skipped tests its reason now names scope partial, the count of skipped tests, and that every test the capture command ran may have been skipped.
  A capture with no skipped tests keeps the old reason.
- A Stuck over such a record, at Drift-Await or at Drift-Baseline's foreground capture, replaces the plain capture way forward with:
  - `lym drift scope --scope full` (or `--scope partial` with a shorter `--skip` list) followed by `lym drift capture` and `lym loom resume`, offered only while HEAD is the branch point;
  - `lym drift scope --scope off` followed by `lym loom resume`;
  - `lym drift defer` followed by `lym loom resume`, as before.
- The stop's reason carries the capture record's reason, so a tackler learns the cause from the stop alone.

## Design decisions

- The ways forward key on the record's `wrote_no_file` flag and its skip list, not on parsing the reason text.
- `lym drift scope` is the parent's verb at a blocked Drift-Baseline or Drift-Await, which is where both Stucks halt; replacing the scope with one that compares a skipped test already drops the stale capture record.
- The capture cannot know that every test was skipped, since it does not read the harness's test list, so the reason says "may have been skipped".

## Tests

- `TestRunCapture` adds a row for exit 0 with no file under skipped tests, checking the scope, count and `wrote_no_file`, and checks the flag and the absence of skip text on the existing rows.
- `TestDriftAwait` adds rows for such a record on and off the branch point.
- `TestDriftBaseline` adds a row for the foreground fallback capture.
