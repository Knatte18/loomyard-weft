---
round: 2
ledger:
  - key: d2-bracket-transcript-unbacked-report-accepted
    rounds: [1]
    status: resolved
  - key: d4-measurement-point-precedes-stencil
    rounds: [1]
    status: resolved
  - key: d1-ambiguity-bound-gitrepo-route
    rounds: [1]
    status: resolved
  - key: d2-settle-window-collapse
    rounds: [1]
    status: resolved
  - key: d2-rebegin-bracket-undefined
    rounds: [1]
    status: resolved
  - key: d4-merriam-session-unspecified
    rounds: [1]
    status: resolved
  - key: d1-nonhex-claim-consume-sites
    rounds: [1]
    status: resolved
  - key: d5-reworded-row-trigger-missing
    rounds: [1]
    status: resolved
  - key: decision-record-deliberation-narrative
    rounds: [1]
    status: resolved
  - key: d1-head-sha-resolution-point-vs-correctness-path
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d4-subtracted-texts-differ-from-merriambase
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised nine findings, including two BLOCKING design ones on D2 and D4.
Round 2 re-raised none of them; its focus verdicts clear each, so all nine are resolved.
Round 2 raised two new MEDIUM findings: where D1 resolves `head_sha` relative to the correctness-failure path, and which texts D4 subtracts from the measured start.
The D4 one is gating-class on a first-raised key, so it converges the loop and Go carries it into `## Open risks`.
