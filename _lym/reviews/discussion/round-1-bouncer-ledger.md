---
round: 1
ledger:
  - key: deliver-confirm-local-sender-resolution
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overlapping-deliveries-stored-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: returned-message-still-confirms-yes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: creel-read-text-shows-no-stamp
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: uncommitted-config-edit-grants-trust
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: operator-text-unverified-mail-override
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: reply-awaiting-outbound-carried-forever
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: receiver-rules-before-duplicate-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-prunes-live-hub-on-read-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: downgrade-loses-outbound-messages
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: confirm-tuple-omits-expects-reply
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: probe-flood-exposure-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risk-sandbox-closed-area-settled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-gaps-d4-d6
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-rationale-relitigates-rejected
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
Fifteen findings are open.
The gating ones are five design findings at MEDIUM, all in the D3 deliver and confirm protocol, D5 trust bound and D6 display.
One decision-class MEDIUM, the departure from the operator's text on unverified mail, waits on the parent.
The rest are LOW or NIT local corrections and bound statements.
