---
round: 2
ledger:
  - key: replay-vs-harvest-auto-accept-outcome
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: soft-hard-cap-measure-undefined
    rounds: [1]
    status: resolved
  - key: step4-returning-key-test-underdefined
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: mechanical-mistag-bound-missing
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: budget-escalation-at-hard-cap-offers-continue
    rounds: [1]
    status: resolved
  - key: hard-cap-escalation-over-recorded-continue-unanswerable
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: orch-stencil-change-has-no-decision
    rounds: [1]
    status: resolved
  - key: rule-continue-cause-unstated
    rounds: [1]
    status: resolved
  - key: hard-cap-bypassed-after-goto-count-reset
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: hard-cap-way-forward-misses-run-settings-layer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1's cap-measure, budget-escalation, orch-stencil and rule-continue-cause findings no longer appear in round 2 and are resolved.
Four round-1 keys reopened in narrower form: the mechanical-tag bound (now false on Darn-Review), the recorded continue at the hard cap (new record or kept record), the earned-CIRCLING step 4 premise (needs the checkpoint), and the auto-accept harvest criterion.
Two new design findings: the hard cap is passed after a goto resets Shed's count (BLOCKING), and the hard-cap way forward misses a task's `run_settings` layer.
