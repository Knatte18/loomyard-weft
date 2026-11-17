---
round: 1
ledger:
  - key: machine-address-split-outside-agentname
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: approve-does-not-deliver-held-in-pass
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: accepted-posts-settled-only-at-relay-start
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bridge-unsettled-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ensurebridge-repeats-listing-every-poll
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rule-constants-repeated-across-packages
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-test-no-per-hub-worker-on-post
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: slackmachine-test-needs-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round: eight findings, all open.
F1 (BLOCKING) is the address split in `crossmachine.go` and `machinemail.go`; F2 (MEDIUM) is the held message left undelivered after approve then revoke in one read.
F3 to F8 are LOW and local.
