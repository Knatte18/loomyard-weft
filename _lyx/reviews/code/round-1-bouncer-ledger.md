---
round: 1
ledger:
  - key: spec-sentence-does-not-name-check
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one NIT: the spec sentence at `contracts/specs/loom-discussion-spec.md:38` says "the format check" instead of naming the discussion check and its `pattern-change-over-cap` finding.
The review text gives the exact replacement wording, so the fix is mechanical.
