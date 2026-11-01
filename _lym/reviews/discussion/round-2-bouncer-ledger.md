---
round: 2
ledger:
  - key: join-dead-runner
    rounds: [1]
    status: resolved
  - key: lease-shortened-by-later-front
    rounds: [1]
    status: resolved
  - key: median-abandoned-call-unmarked
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-verify-runner-slot-phase-cancel
    rounds: [1]
    status: resolved
  - key: runner-verb-unbounded-direct-call
    rounds: [1]
    status: resolved
  - key: lease-write-before-spawn
    rounds: [1]
    status: resolved
  - key: wait-constraint-operator-commands
    rounds: [1]
    status: resolved
  - key: second-front-reruns-gate
    rounds: [1]
    status: resolved
  - key: replay-bound-contradicts-75-rule
    rounds: [1]
    status: resolved
  - key: d9-stencils-hold-no-300000
    rounds: [1]
    status: resolved
  - key: fork-cache-lifetime-no-evidence
    rounds: [1]
    status: resolved
  - key: windows-join-start-time
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: cap-excludes-claim-and-lock-waits
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card-verify-killed-runner-orphans-line
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: late-claim-not-refused
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: gate-report-counts-abandoned
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-verify-join-key-ambiguous
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: runner-description-fits-gate-test-only
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lease-shortening-two-writes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: wall-time-omits-pre-tier-phase
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings; the round 2 review no longer raises ten of them, so those keys are resolved.
`median-abandoned-call-unmarked` stays open: round 2 F4 finds the Abandoned mark still misses the build step's and a blocked tier's records.
Round 2 raised ten findings, nine on new keys.
F1 is BLOCKING: the join test cannot work on Windows without `proc.CreationTime`.
F2 and F3 are MEDIUM design findings, on the cap not covering the claim and lock waits, and on a killed card-verify runner leaving its line running.
F5 to F10 are LOW or NIT; F6 to F10 are local corrections the review text already fixes.
