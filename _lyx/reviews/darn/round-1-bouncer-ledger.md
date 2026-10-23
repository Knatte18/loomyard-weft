---
round: 1
ledger:
  - key: failed-mailreadat-lookup-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pause-discount-after-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: requestreadat-clearing-pinned-at-one-site
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: doc-understates-no-timeout-case
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all test-coverage or doc gaps in the handoff timeout counted from the orch's read of the note request.
The gating one is the untested failed `MailReadAt` lookup.
The other three are LOW: the pause discount after the read, the reset sites that clear `RequestReadAt`, and a doc sentence narrower than the code.
Each fix is fully specified by its finding text.
