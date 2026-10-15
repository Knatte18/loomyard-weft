---
round: 3
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
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-engine-state-not-carried-across-rebegin
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-run-checks-recovery-strand-rule-false
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d6-engine-less-entry-bound
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
---

Round 1's seven findings and round 2's engine-record finding are resolved; none was raised again.
The round 2 wiring-placement finding is back in round 3 as BLOCKING: D6 names verbs but still has no check site that is true of the code, and the review's fix moves the check to the spawn sites (`websterengine.Run` before `StartMaster`, recover-batch's spawn branch before the strand starts).
Round 3 adds three new keys in D6.
The state keys have no stated home and no carry-forward across re-begin and respawn (BLOCKING).
The rule sentence and the `Bound:` sentence say each verb checks only what it spawns, which is false for `run` (MEDIUM).
An engine-less state entry reads as the current default with no bound for a default that audits forks, and a recorded name the engine set does not hold has no outcome (MEDIUM).
