---
round: 1
ledger:
  - key: test-helper-resign-arrows-cards-9-13
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-4-bound-stated-by-no-card
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: bouncer-refusal-row-misstates-outcome
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: wirelightweight-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-languageconfig-line-omits-rename-tools
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none design-class and none BLOCKING.
All five are local corrections whose text the review fixes: missing re-sign arrows on test helpers (F1), a missing bound sentence for D5.4 (F2), a misdescribed Bouncer refusal row (F3), a stale `wireLightweight` doc comment (F4) and a missing `docs/overview.md` line (F5).
