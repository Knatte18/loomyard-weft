---
dispositions:
  - id: F1
    disposition: fixed
---

# Round 1 fixer-report

## Changes

- F1, fixed as proposed, commit `efbb1979`.
  The `handOver` doc in `internal/sheddriver/drive.go` now says a failed spawn leaves the run parked with its report for the parent,
  and that a watching batten re-steps it only over a tree with no uncommitted paths outside the run's records.
- F1, `_lym/landing/summary.md`, Design decisions, third bullet, now says the failure reaches batten's path only when the successor spawn fails, where the parked run waits for the parent.
  The file sits under `_lym`, so it is edited in the working tree and not committed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left other `_lym` and `.md` documents outside `internal` and `_lym/landing` unsearched.

## Disputed

None.

## Deferred

None.

## Rulings

None.
