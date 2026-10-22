---
round: 2
ledger:
  - key: d1-coverage-read-before-idle-wait
    rounds: [1]
    status: resolved
  - key: d6-orch-guid-shape
    rounds: [1]
    status: resolved
  - key: d1-setpause-holds-orch-start
    rounds: [1]
    status: resolved
  - key: d4-verify-starting-under-pause-not-held
    rounds: [1]
    status: resolved
  - key: d4-sigkill-escalation-not-from-waitdelay
    rounds: [1]
    status: resolved
  - key: d4-cancel-change-reaches-other-verifyrun-callers
    rounds: [1]
    status: resolved
  - key: d8-failed-guid-lookup-strands-master
    rounds: [1]
    status: resolved
  - key: d1-fail-open-states-unbounded
    rounds: [1]
    status: resolved
  - key: d1-sendconditioned-bypasses-sendverified
    rounds: [1]
    status: resolved
  - key: d7-reread-does-not-exclude-resume
    rounds: [1]
    status: resolved
  - key: d3-expired-finalize-runs-gate-under-cover
    rounds: [1]
    status: resolved
  - key: d4-activity-readers-misread-paused-verify
    rounds: [1]
    status: resolved
  - key: plan-note-cancel-path-missing-at-two-sites
    rounds: [1]
    status: resolved
  - key: d2-deferred-prompt-not-run-start-send
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-negative-bounds
    rounds: [1]
    status: resolved
  - key: d4-sigterm-cancel-home-unstated
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d2-wait-slug-differs-from-refusal-slug
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-orch-runner-armed-for-unused-read
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-failed-lookup-refusal-lacks-end-turn
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised fifteen findings, one BLOCKING and the rest MEDIUM or LOW.
Round 2 did not raise any of them again and its focus verdicts clear the D1 idle-wait ordering and the orch GUID shape, so all fifteen are resolved.
Round 2 raised four new findings, none gating.
The MEDIUM one is that D4 does not say where the SIGTERM-first cancel lives, with `proc.ConfigureGroupKill` shared by `gitexec` and the `lyxbin` test kit.
The LOW ones are D2's wait and D1's refusal judging different slugs, D1 arming the orch runner for a read none of its lines take, and D8's failed-lookup refusal not telling Master to end its turn.
All four are local text corrections.
