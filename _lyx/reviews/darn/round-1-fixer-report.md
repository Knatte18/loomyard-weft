# Fixer-report, round 1

## Changes

- F1: `TestCountOrch` in `tools/tokencount/orch_test.go` writes message `straddle` on two lines in the prime transcript.
  The first line, at 09:00 outside the span, carries 1 output token; the last, at 12:30 between windows, carries 500,000.
  The expected unattributed charge is now 3.5M, `orch.Main.Messages` is 4, and the report rows for orch, total and unattributed follow.
  A reader of the first line would drop the message, so the test fails on the old behavior.
  `lyx gate test ./tools/tokencount` is green; committed.
- F2: `_lyx/landing/summary.md` now says only the run counter's dropped earlier lines add to the repeated-lines count, and the orch counter does not report its drops.
  The file is edited but not committed, since the round never runs git against `_lyx`.

## Disputed

None.

## Deferred

None.
