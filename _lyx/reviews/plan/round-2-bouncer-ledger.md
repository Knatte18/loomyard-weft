---
round: 2
ledger:
  - key: relaunch-refusals-no-refusal-spec-row
    rounds: [1]
    status: resolved
  - key: driver-wake-rule-drops-d6-condition
    rounds: [1]
    status: resolved
  - key: consume-refusals-no-reaching-test
    rounds: [1]
    status: resolved
  - key: arbiter-verb-unreadable-input-no-test
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: consume-trusts-verdict-not-envelope
    rounds: [1]
    status: resolved
  - key: past-deadline-rerun-unhandled
    rounds: [1]
    status: resolved
  - key: card14-routing-edges-no-bound
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: eligibility-tested-twice
    rounds: [1]
    status: resolved
  - key: consume-test-reasserts-relaunch-effects
    rounds: [1]
    status: resolved
  - key: waiter-refusal-rows-belong-in-existing-test
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: checkgototarget-changes-goto-precedence
    rounds: [1]
    status: resolved
  - key: missing-status-classed-transient
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: rerun-without-marker-abandons-live-arbiter
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised eleven findings; round 2 raised five.
Seven round 1 keys are not raised again and are resolved.
Three carry over in narrower form: the status-read-error test row (F1), the stencil-only half of card 14's bound (F4), and the flag-without-until-stop row placement (F3).
Two keys are new in round 2, both LOW: the transient class of the missing-status row (F2) and the re-run with no marker (F5).
