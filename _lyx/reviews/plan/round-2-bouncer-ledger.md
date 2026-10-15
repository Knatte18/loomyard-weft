---
round: 2
ledger:
  - key: shared-golden-under-package-testdata
    rounds: [1]
    status: resolved
  - key: idless-key-fallback-collides-across-transcripts
    rounds: [1]
    status: resolved
  - key: sessionusage-row-pins-behavior-change
    rounds: [1]
    status: resolved
  - key: card5-resignatures-without-arrow
    rounds: [1]
    status: resolved
  - key: line-fallback-key-unobserved-by-test
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four consistency findings; none was raised again in round 2, so all are resolved.
Round 2 raised one LOW scope finding: the `line:<transcriptKey>:<n>` fallback key is pinned by no test row that can observe it, and `<n>` is undefined.
It is not gating, so the loop converges.
