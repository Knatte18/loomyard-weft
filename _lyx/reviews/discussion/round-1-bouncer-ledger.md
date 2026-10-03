---
round: 1
ledger:
  - key: legacy-alias-settles-at-harvest
    rounds: [1]
    status: resolved
  - key: parse-error-latest-round-no-rule
    rounds: [1]
    status: resolved
  - key: departure-rule-direction-and-blocking
    rounds: [1]
    status: resolved
  - key: accept-pending-vs-settled-state
    rounds: [1]
    status: resolved
  - key: facts-fixture-ledger-range
    rounds: [1]
    status: resolved
  - key: circling-verbs-agent-shell-bound
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: budget-block-reason-in-open-risks
    rounds: [1]
    status: resolved
---

Round 1 raised five MEDIUM and three LOW findings, all against `decision-record.md`.
Four MEDIUMs were bound or state-model gaps on the new routing edges: legacy verdict aliases reachable at harvest (D1), a parse-error latest round with no decision rule (D3/D6), a one-directional departure rule silent on BLOCKING (D6), and an accept with no pending/settled distinction (D7).
The fifth MEDIUM was an acceptance criterion that contradicted D3's ledger range.
The LOWs covered D7's trust-boundary wording, Auto-mode assumptions restating decisions, and a requirement that sat in Open risks.
The fixer addressed all eight in the decision record, and each fix is reflected in the acceptance criteria where it applies.
No finding is open going into round 2.
