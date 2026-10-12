---
round: 2
ledger:
  - key: overlay-canonicalization-writes-and-reloads
    rounds: [1]
    status: resolved
  - key: plan-gate-double-delete-before-reference
    rounds: [1]
    status: resolved
  - key: recover-batch-refusal-undecided
    rounds: [1]
    status: resolved
  - key: candidate-walk-skipped-directories
    rounds: [1]
    status: resolved
  - key: overlay-accepts-non-plan-md
    rounds: [1]
    status: resolved
  - key: missing-acceptance-rows
    rounds: [1]
    status: resolved
  - key: bisection-procedure-unspecified
    rounds: [1]
    status: resolved
  - key: no-enclosing-span-disposition
    rounds: [1]
    status: resolved
  - key: joint-conflict-alongside-solo-failure
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: package-self-glyph-delete-placement
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: overlay-refusals-vs-acceptance-row
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's eight findings were all settled by round 2: the review's focus verdicts mark the overlay writes and reads, the done card at the plan gate, `recover-batch`, the overlay name rule and skipped directories as not defects.
Round 2 raised three LOW findings: Decision 6's search procedure misses a joint conflict alongside a solo failure (F1), Decision 1 does not place the package self glyph Delete check (F2), and Decision 5's refusal list omits two refusals the acceptance row requires (F3).
None is gating.
