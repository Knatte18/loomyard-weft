---
round: 1
ledger:
  - key: continue-pass-skips-owner-reconcile
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pair-teardown-claim-clear-restarts-removed-task
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: innerrun-memory-inventory-deferred
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pass-in-flight-through-inprocess-waits
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reed-down-ends-beam-dobby
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: continue-pass-flag-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: halted-run-resumer-disagrees
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: respawn-cap-spent-in-harmless-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: third-live-batten-reader
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-arbiter-copy-hub-layer-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: landing-count-misses-hand-passes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: older-binaries-write-statusless-tasks
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reexec-versus-first-pass
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: two-dedup-keys-for-dobby-notices
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: backoff-with-only-halted-runs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: late-fswatch-event-after-pass-exit
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d11-rationale-by-rejection
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seventeen findings, mapped one to one to the keys above (F1 to F17 in order).
The gating core is the owner-machine gap after a wake, the claim clear on pair removal, and the deferred InnerRun memory inventory; F4 to F6 are the gating MEDIUM design findings.
Keys marked `design-wrong` change a decision or contract and need a fresh review of the new text; `mechanical` keys are local corrections the finding text already fixes.
