---
round: 1
ledger:
  - key: card4-landingdeps-red-until-card5
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card1-unreadable-file-marked-invalid
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card1-skills-hatch-no-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card8-language-skills-key-no-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card1-local-override-row-unreachable
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none recurring.
Two are gating design findings: card 4 leaves `internal/loomcli` red until card 5, and card 1's `Load` wraps transient read errors as invalid content.
The remaining three are missing attack-surface bounds on the `skills` key (card 1) and the `language-skills` profile key (card 8), and a local-override test row that never reaches the local layer.
