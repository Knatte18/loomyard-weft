---
round: 1
ledger:
  - key: card6-breaks-burlerengine-template-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card3-config-refusals-no-spec-row
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: strandremover-hidden-in-engine-card
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-no-handoff-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card1-probegated-retests-attach
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: step3-header-names-deleted-orchestrator
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: overview-doc-not-updated
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card4-untested-error-exits
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card7-donegone-row-cannot-assert-removal
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the plan; the ledger has no earlier rounds.
Two BLOCKING findings: card 6 leaves `internal/burlerengine/template_test.go` uncompilable, and card 3 adds config refusals with no spec row or way forward.
Two gating design findings: `StrandRemover` hidden inside the engine-split card, and card 6's missing bound for the handoff and moved gates.
The rest are test-economy, doc and wording gaps in cards 1, 4, 5 and 7 and the docs list.
