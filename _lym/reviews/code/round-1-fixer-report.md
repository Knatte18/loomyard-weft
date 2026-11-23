---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `contracts/stencils/harden/harden-rubric-review.md`, the **Converged.** bullet no longer requires that the reviewer model differ from the previous round's.
  It now reads "no BLOCKING or MEDIUM finding and whose live gate is green", and the next line reads "It reads both from the round's facts."
  Committed as `f5e3d79fe`.
  No Go code changed.

## Self-check

- fix-code-correctness: no findings.
  Its open question, whether the stencil's `lym-stencil:` header digest needs a refresh, does not apply: the header sits in the deployed copy, not in the `contracts` source file.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
