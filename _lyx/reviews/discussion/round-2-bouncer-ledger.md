---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-wiring-check-verb-placement
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1's seven findings are not raised again in round 2, so they are resolved.
Round 2 raised two findings, both in D6.
The BLOCKING one says the per-run engine is read from a shuttle run record that is deleted on a done run, so the lookup falls back to the default engine; the fix moves the record into webster's own state.
The MEDIUM one says D6 does not name which verbs run the wiring check, which makes one of its own sentences false either way; the fix names the roles each verb checks.
