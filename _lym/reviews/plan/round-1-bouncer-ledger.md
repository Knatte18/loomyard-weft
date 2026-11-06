---
round: 1
ledger:
  - key: card9-reply-guard-breaks-card7-drop-row
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: card8-socket-text-bypasses-notice-quoting
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card9-reportengine-doc-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card1-way-forward-overclaims-verbs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-overview-creel-bullet-verb-list
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card9-storeinboundlocked-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings.
F1 (BLOCKING) is a cross-card test conflict: card 9's reply guard refuses the reply-to case of card 7's drop row before the confirm.
F2 (MEDIUM, design) is a gap in card 8's mapping of socket refusals to quoted notice text.
F3 to F6 are listing and wording corrections on cards 1, 5 and 9.
No earlier ledger exists.
