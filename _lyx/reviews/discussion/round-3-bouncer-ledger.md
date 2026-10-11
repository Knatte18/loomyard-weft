---
round: 3
ledger:
  - key: d4-d5-guard-exit-status-conflated
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: premise-client-queue-never-yields
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: interrupted-starter-leaves-fileless-server
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: d5-exit-unattached-kills-server-before-pin
    rounds: [2]
    status: resolved
  - key: d5-destroy-unattached-kills-session-before-pin
    rounds: [1, 2]
    status: resolved
  - key: d3-server-starting-boot-undefined
    rounds: [1, 2]
    status: resolved
  - key: d2-glob-metacharacters-in-path
    rounds: [1, 2]
    status: resolved
  - key: foreign-sessions-keep-server-alive
    rounds: [1, 2]
    status: resolved
  - key: d4-parse-error-applies-nothing
    rounds: [1]
    status: resolved
  - key: d5-open-ended-pin-clause
    rounds: [1]
    status: resolved
  - key: tmux-conf-absence-bullet-misfiled
    rounds: [1]
    status: resolved
---

Rounds 1 and 2 worked the claim, pin ordering and path-rule defects; round 3's focus verdicts mark pin sequencing, the spawn-to-pin window, the claim command string and the session-listing surfaces settled, so those keys are resolved.
Moving `source-file` into the guard invocation to fix the pin ordering created the open BLOCKING defect: one tmux exit status now carries both the source result (D4: warn) and the pin result (D5: fatal), so an operator-file typo fails the boot.
Two LOW findings remain: the "one client's queue runs to completion" premise fails when the sourced file holds a waiting command, and a starter interrupted before its guard invocation leaves a file-less server that D3 does not bound.
