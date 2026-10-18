---
round: 2
ledger:
  - key: d4-no-route-after-landing
    rounds: [1]
    status: resolved
  - key: d4-stop-removes-own-orch
    rounds: [1]
    status: resolved
  - key: d3-restore-on-failed-commit
    rounds: [1]
    status: resolved
  - key: d1-prime-root-field-dropped
    rounds: [1]
    status: resolved
  - key: d3-whole-file-replace-deletes-rules
    rounds: [1]
    status: resolved
  - key: d3-no-pull-first-write
    rounds: [1]
    status: resolved
  - key: d3-unchanged-content-push-undecided
    rounds: [1]
    status: resolved
  - key: d3-prime-only-bound-misstated
    rounds: [1]
    status: resolved
  - key: d3-temp-file-second-bolt-path
    rounds: [1]
    status: resolved
  - key: d2-heading-level-rule-no-carrier
    rounds: [1]
    status: resolved
  - key: d4-bolt-premise-false-reminder-gap
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-failed-commit-recovery-is-sweep
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-restore-in-closure-dead-path
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-heading-rule-stated-twice
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

All ten round-1 keys were not re-raised in round 2 and are resolved.
The round-2 review confirms D4's operator route holds on every landing path and that the D1, D3 bound and temporary-file points hold.

Round 2 raised four new keys.
The one with weight is `d4-bolt-premise-false-reminder-gap`: D4 claims a card cannot write Bolt, but `lyx board` does, so a board entry could carry the post-landing step and close the no-reminder risk.
`d3-failed-commit-recovery-is-sweep` and `d3-restore-in-closure-dead-path` are D3 corrections, and the heading rule stated twice in D2 is a NIT.
