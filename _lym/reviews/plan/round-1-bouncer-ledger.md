---
round: 1
ledger:
  - key: reportengine-allowlist-misses-card11-imports
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: chase-timer-never-backs-off
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reconnecting-wait-reply-misses-landed-reply
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: creel-sender-notice-makes-creel-a-receiver
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-drops-without-added-after-discussion
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: expect-reply-flag-states-no-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: linefits-reaches-noticefits-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: daemon-wakeups-sweep-bullet-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seat-smoke-test-comments-name-sendmessage
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readers-carries-no-day-for-retention
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: store-mail-cannot-observe-replied
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card2-resolution-rows-repeat-walk-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1, first ledger.
F1 to F12 map in order to the keys above.
The gating set is F1 and F2 at BLOCKING, plus F3 and F4 at MEDIUM design.
F2, F3, F4 and F5 change a design or a recorded decision, so their fixes need a fresh review; the rest are local corrections the finding text fixes.
