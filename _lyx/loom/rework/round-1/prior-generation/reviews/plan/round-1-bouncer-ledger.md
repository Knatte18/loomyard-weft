---
round: 1
ledger:
  - key: hub-delivery-test-untagged-tier
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: creel-cli-test-untagged-tier
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: removed-holder-sweep-no-backoff
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card7-depends-on-later-cards
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: withheld-orch-reminder-no-event
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: reconnected-waiter-reply-no-row
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: inject-privilege-bound-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: strand-caller-rows-linux-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: background-overstates-import-scan
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, none carried from an earlier round.
Two BLOCKING test-tier findings (F1, F2) ask for integration tags on `delivery_test.go` and `cli_test.go`.
The BLOCKING sweep finding (F3) needs a backoff on card 9's 30-second removed-holder sweep, a change to the answer key's fixed interval.
F4 moves the waiter registry into card 7 so card 7 stops depending on cards 9 and 10.
F5 adds a withheld journal event to card 6 for the orch's reminders.
F6 to F9 are local gaps: a reconnect row, the inject bound, Windows strand identity, and the import-scan wording.
