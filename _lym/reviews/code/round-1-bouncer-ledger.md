---
round: 1
ledger:
  - key: vstest-result-words-reach-every-args-tier
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: measurecall-tier-fallback-conflicts-with-pattern
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: shell-spawning-tests-in-tier0
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: measurecall-renders-refused-call
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: env-line-role-clause-failed-instruction
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: env-line-prints-session-value
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: trx-test-leaves-baseline-and-fallbacks-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: session-env-capture-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-wiring-gate-config-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: caller-result-options-take-trx-away
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: warning-run-messages-become-causes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pending-console-line-read-after-replacement
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-args-tier-gets-no-crash-cause-undocumented
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: env-line-repeats-under-copies
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, with no prior ledger.
Its keys map to the review's findings in order: F1 to F14.
F1 and F2 are the design-level calls: whether the result words need an explicit opt-in tier setting, and how the measuring call's `--tier` fallback squares with the PATTERN gate-key rules.
F2 belongs to the run's parent, since only it can record the call with `lym loom decision add`.
F4 reshapes `MeasureCall`'s contract, so it is marked `design-wrong`; the rest are local corrections the finding text already fixes.
