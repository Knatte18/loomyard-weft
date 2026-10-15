---
round: 3
ledger:
  - key: hub-delivery-test-untagged-tier
    rounds: [1]
    status: resolved
  - key: creel-cli-test-untagged-tier
    rounds: [1]
    status: resolved
  - key: removed-holder-sweep-no-backoff
    rounds: [1]
    status: resolved
  - key: card7-depends-on-later-cards
    rounds: [1]
    status: resolved
  - key: withheld-orch-reminder-no-event
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: reconnected-waiter-reply-no-row
    rounds: [1]
    status: resolved
  - key: inject-privilege-bound-missing
    rounds: [1, 2]
    status: resolved
  - key: strand-caller-rows-linux-only
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: background-overstates-import-scan
    rounds: [1]
    status: resolved
  - key: delivery-worker-no-backoff-when-idle-or-dormant
    rounds: [2]
    status: resolved
  - key: interrupt-checks-split-across-cards-7-and-9
    rounds: [2]
    status: resolved
  - key: creelfake-clock-records-no-waits
    rounds: [2]
    status: resolved
  - key: creelfake-tests-need-external-test-package
    rounds: [2]
    status: resolved
  - key: idle-paused-errors-unhandled
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reply-does-not-wake-scheduler
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: inject-interrupt-failed-outcome-open
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: what-counts-omits-reset-events
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Rounds 1 and 2 raised sixteen findings between them, three of them BLOCKING in round 1 and one in round 2.
Round 3 raised six and none is BLOCKING or design-class.
The round 2 design BLOCKING on the delivery worker's backoff, the interrupt-check split, the inject Bound and the creelfake findings are resolved: the review's Rejected section clears each.
Two older keys reappear in narrower form: the orch withheld event now lacks a sweep test row (F3), and the strand pane pid for EXTERNAL rows is unspecified (F4).
Four keys are new: Idle and Paused error handling (F1), the reply wake (F2), the failed-interrupt outcome of inject (F5) and the reset events in the What-counts entry (F6).
