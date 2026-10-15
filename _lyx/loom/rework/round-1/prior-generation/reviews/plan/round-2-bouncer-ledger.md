---
round: 2
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
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reconnected-waiter-reply-no-row
    rounds: [1]
    status: resolved
  - key: inject-privilege-bound-missing
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: strand-caller-rows-linux-only
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: background-overstates-import-scan
    rounds: [1]
    status: resolved
  - key: delivery-worker-no-backoff-when-idle-or-dormant
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: interrupt-checks-split-across-cards-7-and-9
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: creelfake-clock-records-no-waits
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: creelfake-tests-need-external-test-package
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; round 2 raised seven.
Six of round 1's keys are resolved: the two test-tier findings, the sweep backoff, card order, the reconnect rows and the import-scan wording.
Three reopened in new form: the orch withheld event now misses the reply path, the inject Bound now overstates the TTL, and the strand identity now rests on unnamed process environment in `TestCreelCLI`.
Round 2 adds a BLOCKING design finding on the delivery worker's backoff, which leaves a dormant receiver or an idle one inside its reminder backoff probed every second.
It also adds the split of the interrupt checks between cards 7 and 9, the missing wait record on `creelfake.Clock`, and the external test package that `creelfake` tests need.
