---
round: 1
ledger:
  - key: card8-new-refuses-darn-before-card11
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: description-commit-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: batten-nillegal-discussionhistory
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: perchhardcap-ignores-tapestry-yaml
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card6-missed-moved-key-readers
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card9-stale-docs-preflight-checkseed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: gate-slot-background-stale-validators
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: not-committed-refusal-darn-write-in-flight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card19-beam-refusal-departs-d2-d11
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card11-reportengine-doc-darn-setup
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: batchercheck-cancel-row-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card5-loomcli-doc-recipe-override
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card11-wrong-card-cite
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
The review raised 13 findings, three of them gating: the card 8 ordering break against card 11 (BLOCKING), the plan perch's hard cap ignoring `tapestry.yaml`, and the Not-committed way forward for a run parked at Darn-Write.
F1, F8 and F9 change a contract or a decided way forward, so they are `design-wrong` and need a fresh review of the new text.
The rest are local corrections whose result the finding text fixes.
