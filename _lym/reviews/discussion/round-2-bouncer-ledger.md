---
round: 2
ledger:
  - key: mode-check-research-deferred
    rounds: [1]
    status: resolved
  - key: operator-stop-copy-clamped-5-lines
    rounds: [1]
    status: resolved
  - key: version-risk-no-trigger
    rounds: [1]
    status: resolved
  - key: ring-wake-operator-copy-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ring-answer-path-early-exit-release
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ack-journaled-before-confirmation
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: only-overstates-ring-answered
    rounds: [1]
    status: resolved
  - key: d5-journal-failure-no-outcome
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings; round 2 raised four.
Four round-1 keys were not raised again and are resolved: the mode-check research, the Stop copy clamp, the version risk trigger and the `only` wording.

The ring wake's operator copy is the one gating key still open.
Round 1 filed it at LOW; round 2 reopens it at MEDIUM because whether the terminal draws a ring wake's body is still left to the smoke run, with a halt rule handed downstream.
The two LOW design keys concern the ring's acknowledgement path: exit 0 against exit 2 on an unconfirmed acknowledgement, and the client's reconnect-and-resume defeating "never resumes the wait".
The one new key is a NIT: D5 has no outcome for a claim acknowledged by the client but not journaled by the daemon.
