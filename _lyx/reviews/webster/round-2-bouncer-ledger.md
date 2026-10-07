---
round: 2
ledger:
  - key: hard-wrapped-new-comments
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: held-run-tests-not-parallel
    rounds: [1]
    status: resolved
  - key: stale-asking-outcome-prose
    rounds: [1]
    status: resolved
  - key: session-continued-in-corpus-duplicate
    rounds: [1]
    status: resolved
  - key: send-tail-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: dead-finalize-message-plumbing
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: diedmaster-call-sites-not-gofmt
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 2 maps F1 to `hard-wrapped-new-comments`, F2 to `send-tail-untested`, F3 to `dead-finalize-message-plumbing` and F4 to `diedmaster-call-sites-not-gofmt`.
The round-1 key `hard-wrapped-new-comments` recurs: the fix reflowed an enumerated location list, and the fresh review found more diff-added comment sentences still hard-wrapped.
It is now class `design`, since the method of enumerating locations by eye is what fails.
The other three round-1 keys were not raised by the fresh round-2 review and are resolved.
