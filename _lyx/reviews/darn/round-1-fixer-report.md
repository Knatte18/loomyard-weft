# Fixer-report, round 1

## Changes

- F1 (LOW), fixed as proposed, commit `d9aa3f33`.
  In `internal/loomcli/checklist_test.go`, the first span of the `plan` gate entry in the row "a visit with spans of two entries draws one gate line per entry in first-appearance order…" now starts at `2026-01-02T12:00:00+02:00`, the same instant as 10:00Z.
  The expected gate line stays `10:00 gate plan 12s refused 2 rework 3m05s`, so removing `.UTC()` at `internal/loomcli/checklist.go:707` renders `12:00` and fails the row.
  No new test function.
  `lyx gate test ./internal/loomcli` is green.

## Self-check

All four forks returned.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
