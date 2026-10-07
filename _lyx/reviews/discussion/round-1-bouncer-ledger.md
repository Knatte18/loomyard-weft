---
round: 1
ledger:
  - key: fabric-reference-scope-no-decision
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d4-over-budget-lone-card-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-fit-undefined-on-thin-data
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: wrong-position-consequence-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d11-one-profile-one-count-assumption
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: review-file-format-owner-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d12-rubric-paraphrases-rule
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d12-wrong-enforcement-component
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: fork-count-disagrees-with-table
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: sed-note-belongs-nowhere
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 is the first review, so every key is new.
F1 is the only BLOCKING finding: the folded-in fabric-reference scope lacks decisions, acceptance criteria and bounds.
The other gating-class findings are D4's unbounded over-budget lone card (F2) and D11's one-profile assumption (F5).
The remaining findings are D9 thin-data fit (F3), the unnamed review-file owner (F6), D12's paraphrasing rubric (F7), and LOW or NIT cleanups (F4, F8 to F11).
