---
round: 4
ledger:
  - key: d6-recovery-strand-fork-audit
    rounds: [1]
    status: resolved
  - key: d6-lookup-refusal-way-forward
    rounds: [1]
    status: resolved
  - key: d8-burler-run-keyless-refusal
    rounds: [1]
    status: resolved
  - key: d5-driver-peer-messaging-vs-out
    rounds: [1]
    status: resolved
  - key: d2-scan-misses-engine-dropping-mapping
    rounds: [1]
    status: resolved
  - key: usage-criterion-narrower-than-d4
    rounds: [1]
    status: resolved
  - key: dead-pointer-to-support-log
    rounds: [1]
    status: resolved
  - key: d6-engine-record-deleted-after-done-run
    rounds: [2]
    status: resolved
  - key: d6-wiring-check-verb-placement
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: d6-engine-state-not-carried-across-rebegin
    rounds: [3]
    status: resolved
  - key: d6-run-checks-recovery-strand-rule-false
    rounds: [3]
    status: resolved
  - key: d6-engine-less-entry-bound
    rounds: [3, 4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-spawn-check-after-mutation
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d6-spawn-site-framing
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-recovery-turn-end-key
    rounds: [4]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Rounds 1 to 3 resolved the early D5, D6, D8 and usage-criterion findings, the re-begin carry-forward and the run-checks rule sentence.
Round 4 reopens the wiring theme: D7's last sentence refuses an unregistered Master engine at wiring, which contradicts D6's rule that verbs spawning nothing are never refused by a key (BLOCKING, consistency).
It also finds the spawn checks placed after state-mutating steps in `Run` and in `recoverSpawn` (MEDIUM), the "spawn site" framing overstated for `run`'s recovery pre-flight (LOW), the recovery turn-end read lacking a stated `recoveryEngines` key (LOW), and the wrong-default Bound's failure mode unsupported where the engine errors on a missing parent transcript (LOW, design, on the engine-less-entry key).
