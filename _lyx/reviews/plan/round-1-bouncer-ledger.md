---
round: 1
ledger:
  - key: chair-include-breaks-single-agent-composer
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: seat-runner-gate-once-own-card
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: seat-runner-gate-once-double-eval-and-blast
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: advisors-mapping-refusal-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d4-deviations-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: seatengine-resigns-vs-additive-constraint
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: stale-discussionspec-references
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: reconcile-test-duplicates-mechanism
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eight findings, all open.
The BLOCKING one is card 4's chair include landing while the single-agent composer is still live until card 7.
The other gating one is the `SeatRunner` gate-once change, which wants its own earlier card.
The rest concern the refusal way forward for a mapping value, unrecorded D4 deviations, signature changes under the additive-only constraint, stale comments after card 7, and a redundant new reconcile test.
