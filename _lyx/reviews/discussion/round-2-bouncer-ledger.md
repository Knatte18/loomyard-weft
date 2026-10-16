---
round: 2
ledger:
  - key: pattern-declaration-unbounded-guard-downgrade
    rounds: [1]
    status: resolved
  - key: verify-fix-departure-bypasses-go-route
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: uncaused-failure-bound-false-verify-premise
    rounds: [1]
    status: resolved
  - key: spawn-refusal-names-diagnostic-not-way-forward
    rounds: [1]
    status: resolved
  - key: no-rewrite-tool-exception-wording
    rounds: [1]
    status: resolved
  - key: scope-out-carries-requirement-and-narrative
    rounds: [1]
    status: resolved
  - key: malformed-record-way-forward-unreachable-for-orch
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pattern-change-after-plan-gate-no-landing-source
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1's guard-bound, uncaused-failure, refusal-wording, comment-wording and scope-out findings no longer appear in round 2, and the review confirms those fixes hold.
The verify-fix departure route is still wrong: the Master writes the summary before the gate walks the fixer's commits, so the data must come from Go writing `VerifyGateNotes` after the wait.
Two new findings: D5.4's malformed-record way forward names the parent, who may not edit the record (BLOCKING), and a PATTERN bullet appended after the plan gate has no stated landing-check source (MEDIUM).
