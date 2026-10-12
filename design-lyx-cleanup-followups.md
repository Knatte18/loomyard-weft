# lyx cleanup follow-ups: concurrent sweep flake, missing refusal tests, testtiming doc

`lyx-cleanup-followups` · Note · bug, batten

## Seen (2026-10-12, PR #574 of `test-tmp-cleanup`)

The PR's own "To check" list left these open after the task landed:

- `TestSweep/two_concurrent_sweeps_both_succeed` in `internal/lyxtemp` failed once and passed on rerun: review the concurrent-sweep path for a race, then fix the code or the test.
- `lyx cleanup` refuses outside the prime, but no test pins the refusal.
- `lyx batten cleanup` skips and reports a deleted run directory without a `status.json` instead of committing it, but no test pins that.
- The `testtiming` doc lacks the resign arrow.

## Idea

Fix the race if there is one, and add the missing tests and the doc arrow in the same change.
Small: a candidate for a bundle with other test or cleanup work.
