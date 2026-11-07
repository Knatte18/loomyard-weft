---
round: 1
ledger:
  - key: midmerge-parked-test-not-on-clashlayer
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stale-arrange-comment-in-cleanup-expect
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings, F1 (scope) and F2 (consistency).
F1 maps to `midmerge-parked-test-not-on-clashlayer`: one test still repeats the `clashLayer` steps by hand.
F2 maps to `stale-arrange-comment-in-cleanup-expect`: a comment still says Arrange where the step now runs in Build.
Both fixes are local and fixed by the finding text.
Neither is gating, so the loop converges.
