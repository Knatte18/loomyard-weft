---
round: 1
ledger:
  - key: redundant-planned-tests
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: outcome-part-merge-unspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: follow-never-ends-on-rewritten-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: worker-follow-verdict-reachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-long-text-over-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-falsified-text-set
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: integration-test-forced-change-undescribed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dead-outputs-and-constants
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fabricengine-scratch-docs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: standing-move-refusal-unplaced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card2-test-sentence-wrong-recorder
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; there is no earlier ledger.
The BLOCKING finding is test economy in cards 7 and 8.
The gating MEDIUM design findings are the unspecified outcome-part merge, the endless follow, the reachable worker follow verdict,
the `Long` text cap and the partly stale falsified text set.
The remaining findings are consistency items and two LOW or NIT corrections.
