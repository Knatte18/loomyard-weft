---
round: 1
ledger:
  - key: card4-history-order-depends-on-card5
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: checklist-resign-arrows-missing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card19-twill-resign-arrows-caller-uncovered
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: twill-burler-yaml-unreadable-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: shedbuild-doc-single-llm-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card17-test-file-paths-instead-of-member-glyphs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none recurring.
The one gating finding is the card 4 and card 5 ordering of the history-order visit record.
The other five are local corrections: re-sign arrows, a missing refusal test, a stale package doc and test-file targets that should be member glyphs.
