---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

## Changes

- F1: added `//testtiming:keep` with a reason above `TestRows_SteersOpenWithWholeCommandRefusal` in `internal/shellguard/rows_test.go`, after a bare `//` line as sibling kept tests have it.
- F2: reworded the doc comments on `pythonRoutes`, `shellEditRefusal` and `headlessClaudeRefusal` in `internal/shellguard/rows.go` to say each opens the row's own steer text, which follows the shared whole-command clause, split over two lines.
- Each fix is its own commit; the two follow-ups from the self-check are folded into a third commit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, bare `//` line missing before the keep marker: fixed as proposed.
- fix-proposal, LOW, rewritten comments past 120 characters: fixed as proposed, split at the sentence boundary.

## Disputed

None.

## Deferred

None.

## Rulings

None.
