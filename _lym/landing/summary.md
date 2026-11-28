# Show each tier's slot wait beside its wall time in the gate summary line

A `lym gate test` call's own duration includes the wait for gate slots,
which under contention can dominate it (300 s was seen in one run).
Before this change the call's output showed only the run's wall time per tier,
so a caller who wanted the wait or the run time without it had to read the run records under the gate directory.

## What changes

- `gateslot.TierSummaryLine`, the stderr line printed after each tier, now carries `wait <s>s` before `wall <s>s`:
  `gate test: tier1 diff discovered 3 passed 2 failed 1 skipped 0 wait 300.0s wall 12.5s exit 1 log ...`.
  `wait` is the time from the start of the slot wait to the start of the run; `wall` is the run's own time, as before.
  A carried tier and a rerun show `wait 0.0s`.
- The line's two readers, the failed-tier and discovered-count patterns in `internal/gateslot/runlog.go`, match the new field.
- `RunRecord.SlotWaitSeconds` computes the wait from the record.
  `lym gate report` now uses it for its `wait_seconds` and `abandoned_wait_seconds` totals instead of computing the same figure inline,
  so the report and the summary line always agree.
- The `lym gate test` help and the `gatecli` package doc say that `wait` and `wall` appear separately on each summary line
  and that `wall` is the run time without the wait.

## Design decisions

- The wait goes on the per-tier summary line rather than in a new stdout field.
  A successful call prints no JSON envelope of its own, and the slot wait is per tier, since each tier takes its own slots.
- The build step's summary line is unchanged: the build runs inside the first opted-in tier's slots,
  so the tier's record already holds that wait.
- The patterns require the new field rather than treating it as optional.
  The writer and readers of the line ship in the same binary, so no older line format needs to parse.

## Tests

`TestTierSummaryLine` in `internal/gateslot/runlog_test.go` pins the exact line for a run that waited 300 s and ran 12.5 s,
and for a carried tier with no wait.
It also checks that `FailedTier` and `TierDiscovered` still read the line.

## Merge danger

**Door:** two-way
**Blast radius:** gate
