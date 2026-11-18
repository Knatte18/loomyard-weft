---
dispositions:
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changes

- F7: `internal/driftengine/doc.go` no longer describes the verb's refusal.
  The two lines were replaced by one that states only driftengine's behavior;
  the self-check then reworded it (see Self-check).
  Commits `e35e9d66` and the follow-up reword.
- F8: `internal/loomcli/driftwiring_test.go` carries a `//testtiming:keep` line with the review's reason, below the doc comment and directly above `func TestDriftDeps_CarryTheDecisionRecord`.
  Commit `4a9124c6`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW on F7's new sentence, which named the plan amendment record and PATTERN changes section that no other line of the package doc defines; fixed as proposed, the line now reads "Any other change to the record, and any change to a file outside it, is not exempt and is named as a change."
- fix-affected-set: no findings; its LOW note on lines 275 and 282 is the exemption's own contract, so no change.

## Disputed

None.

## Deferred

None.

## Rulings

None.
