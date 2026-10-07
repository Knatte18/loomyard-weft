---
round: 3
ledger:
  - key: hard-wrapped-new-comments
    rounds: [1, 2, 3]
    status: open
    class: consistency
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
    status: resolved
  - key: dead-finalize-message-plumbing
    rounds: [2]
    status: resolved
  - key: diedmaster-call-sites-not-gofmt
    rounds: [2]
    status: resolved
---

Round 3 maps F1 to `hard-wrapped-new-comments`.
The key has been open in every round: round 1 and round 2 each found hard-wrapped comment sentences the previous fix missed, and round 3 found five more sites with a mid-phrase or fragment break.
The round-3 review files it as `consistency`, and this ledger follows that class, since F1's entry shows no mislabel to justify a relabel.
The three round-2 keys `send-tail-untested`, `dead-finalize-message-plumbing` and `diedmaster-call-sites-not-gofmt` were fixed and are not raised again, so they are resolved.
The three round-1 keys stay resolved.
