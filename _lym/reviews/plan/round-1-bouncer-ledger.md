---
round: 1
ledger:
  - key: card7-store-failure-departs-from-d6
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card2-post-save-removal-error
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pastapproval-error-branches-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: copyrecordtoboard-notfound-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rename-adopts-orphan-record-file
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: refused-rename-row-lacks-validateslug-seam
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: boardcli-doc-payload-rule-omits-discussed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card7-leaves-two-comments-untrue
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-misstates-config-board-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, none BLOCKING.
F2 (`card2-post-save-removal-error`) is the only gating one: design at MEDIUM.
F1 (`card7-store-failure-departs-from-d6`) is MEDIUM but class decision, so it does not gate.
F3 and F4 are unreachable-branch findings in cards 8 and 9; F5 is a rename edge in card 2.
F6 to F9 are consistency fixes to tests, a package doc and stale comments.
No key recurs, since this is the first round.
