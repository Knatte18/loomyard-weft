---
round: 2
ledger:
  - key: hub-pause-caller-unbounded
    rounds: [1]
    status: resolved
  - key: parent-lock-exception-unbounded
    rounds: [1]
    status: resolved
  - key: covered-lock-holder-freezes-others
    rounds: [1]
    status: resolved
  - key: orch-interrupt-skips-grace-and-report
    rounds: [1]
    status: resolved
  - key: orch-resume-line-ownership
    rounds: [1]
    status: resolved
  - key: loom-lock-queue-no-way-out
    rounds: [1]
    status: resolved
  - key: post-kill-stop-unbounded
    rounds: [1]
    status: resolved
  - key: continue-line-no-prewrite-guard
    rounds: [1]
    status: resolved
  - key: goal-overstates-boundary-pause-removal
    rounds: [1]
    status: resolved
  - key: d4-rearg-rejected-interrupt-alternative
    rounds: [1]
    status: resolved
  - key: question-ledger-stale-open-risks-pointer
    rounds: [1]
    status: resolved
  - key: covered-waiter-blocks-parent-lock-queue
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d10-shuttle-tie-names-absent-field
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-drops-grace-for-busy-orch
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: agent-refusal-sends-agent-to-operator
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: caller-bound-rests-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: hub-held-operator-commands-wait-silently
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: boundary-pause-after-said-unchanged
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: support-log-contradicts-d1-caller-rule
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: stale-no-rounds-yet-seed-line
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, led by three BLOCKING design findings on unbounded new capability.
Round 2's review raised none of those keys again, so all eleven are resolved.
Round 2 raised nine new findings, none BLOCKING.
The gating pair is a covered waiter holding the parent lock queue and D10's shuttle-run tie naming a field the record lacks; both are first raised in round 2 and go to Open risks.
The rest are wording and consistency fixes.
