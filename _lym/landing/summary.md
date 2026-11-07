# Re-measure tier 1 and internal/fabricengine after the speed tasks

Adds `docs/research/speed/fabric-test-speed-remeasure-2026-11-07.md`, the re-measure the fabric test speed tasks were waiting on.
It touches no code.

## Results

- A whole `lym gate test --tier 1 --all -- -count=1` pass took 65.4 s against W1's 75.0 s and the expected 56 s.
- `internal/fabricengine` took 47.7 s inside that pass and 41.5 s alone, against W1's 57.0 s and the expected 36 s.
  It is still above the 35 s at which the other packages set the wall.
- The hub's tier commands do not carry `{test_targets}`, so the 48 s expectation does not apply.

## Design decisions

- Neve was not quiet: the one-minute load was 3.34 before the whole pass and 6.36 before the package pass, with interactive sessions running.
  The report states this and reads the figures as high against W1, by an unmeasured amount.
- The package command as the record words it ran nothing, because the gate's green record carried the package over the unchanged tree.
  The measured pass added `-shuffle=off`, go test's default order, to defeat the carry.
- The report holds the measurement and no board note.
  The decision record's parent entry, "The writer runs both timed passes itself", sets both: the writer runs the passes itself, and files no board note in place of the measurement.
- `lym beam status` refuses in a task worktree, so the load comes from `/proc/loadavg`, and the runs in flight from `lym loom peers` and `ps`.

## Open

A re-run with no other session active would separate load from a shortfall against the 36 s expectation.
