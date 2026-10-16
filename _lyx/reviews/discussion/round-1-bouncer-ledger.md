---
round: 1
ledger:
  - key: d6-recovery-strand-fork-audit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-lookup-refusal-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d8-burler-run-keyless-refusal
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-driver-peer-messaging-vs-out
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-scan-misses-engine-dropping-mapping
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: usage-criterion-narrower-than-d4
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: dead-pointer-to-support-log
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all open.
The gating four sit in D2, D6 and D8: D6's checks miss the recovery strand's engine and give the lookup refusal a way forward that cannot clear it, D8 does not cover the keyless `lyx burler run` caller, and D2's scan does not catch a hand mapping that drops the engine.
The other three, in D5 and two acceptance or Auto-mode lines, are local corrections the finding text already fixes.
