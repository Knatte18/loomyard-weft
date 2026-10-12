---
round: 1
ledger:
  - key: relaunch-refusals-no-refusal-spec-row
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: driver-wake-rule-drops-d6-condition
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: consume-refusals-no-reaching-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: arbiter-verb-unreadable-input-no-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: consume-trusts-verdict-not-envelope
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: past-deadline-rerun-unhandled
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card14-routing-edges-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: eligibility-tested-twice
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: consume-test-reasserts-relaunch-effects
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: waiter-refusal-rows-belong-in-existing-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: checkgototarget-changes-goto-precedence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round, first ledger: eleven findings, all raised in round 1.
Four BLOCKING: missing refusal-spec row for Relaunch, a driver wake rule broader than its decision, and two refusals with no reaching test.
Two gating design findings: consumption reads the agent's verdict instead of the verb's validated envelope, and card 14 states no bound.
The rest are test-economy and precedence consistency items plus a past-deadline re-run gap.
