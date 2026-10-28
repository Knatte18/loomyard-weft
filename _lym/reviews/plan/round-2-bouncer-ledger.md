---
round: 2
ledger:
  - key: reportengine-allowlist-misses-card11-imports
    rounds: [1]
    status: resolved
  - key: chase-timer-never-backs-off
    rounds: [1]
    status: resolved
  - key: reconnecting-wait-reply-misses-landed-reply
    rounds: [1]
    status: resolved
  - key: creel-sender-notice-makes-creel-a-receiver
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d10-drops-without-added-after-discussion
    rounds: [1]
    status: resolved
  - key: expect-reply-flag-states-no-bound
    rounds: [1]
    status: resolved
  - key: linefits-reaches-noticefits-unlisted
    rounds: [1]
    status: resolved
  - key: daemon-wakeups-sweep-bullet-stale
    rounds: [1]
    status: resolved
  - key: seat-smoke-test-comments-name-sendmessage
    rounds: [1]
    status: resolved
  - key: readers-carries-no-day-for-retention
    rounds: [1]
    status: resolved
  - key: store-mail-cannot-observe-replied
    rounds: [1]
    status: resolved
  - key: card2-resolution-rows-repeat-walk-tests
    rounds: [1]
    status: resolved
  - key: chaser-first-tick-has-no-previous-reading
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 raised twelve findings; round 2 raised two.
Eleven round-1 keys are resolved: round 2's review does not raise them again and its Rejected section confirms the chase cadence, reply-wait reconnect, reportengine allowlist and D10 refusal fixes.

F1 maps to `creel-sender-notice-makes-creel-a-receiver`, reopened: the `applyPosted` path is closed, the `applyCarried` retention path is not.
Its fix adds a clause and a test row whose result the finding text fixes, so it is `mechanical`.

F2 is the new key `chaser-first-tick-has-no-previous-reading`.
Its fix picks a charging rule that changes what the cadence rows assert, so it needs a fresh review.
