---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `_lym/landing/summary.md`, the net-lines-removed cell of the `internal/shuttleengine` row changed from 6 to 5, matching `git diff --numstat 9c5bd131..HEAD` (8 removed, 3 added).
  No commit was made: the summary lies under `_lym`, and this round's rules forbid any git command against a `_lym` path.
  The change stays in the working tree for the loop that owns `_lym`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
