---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

- F1 (`68dd8f6f8`): `parentReviewAskVerb` refuses when the standing is satisfied, writing nothing, with the way forward `lym loom pr-continue`.
  The verb's doc comment and the ask line in `internal/loomcli/doc.go` carry the clause.
  A `TestParentReviewAskVerb` row records a satisfied PR-Review ask followed by an open one and asserts the refusal text and that no ask is added.
- F2, F3 (`56ab18119`): in the autonomous PR-Review stencil, the satisfied-stands line moved after "It ends the review", and the await-outcome rule opens with the pr-continue exception.
- F4 (`98d088da2`): the `PRReviewAwaiting` doc names the satisfied precedence.

A follow-up commit repaired the gate's comment findings: the `PRReviewAwaiting` doc and the `doc.go` ask clause now break one sentence per line, none over 120 characters.

`lym gate test ./internal/loomcli ./internal/landingshed` passed before that repair.

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
