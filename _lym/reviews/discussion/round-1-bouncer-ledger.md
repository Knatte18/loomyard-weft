---
round: 1
ledger:
  - key: watch-memory-deleted-for-done-run
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landed-cancelled-skip-misfires
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: absent-child-part-reads-as-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: record-removing-cancel-unreported
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stop-change-rule-d2-vs-d3
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: watch-held-with-scheduling-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: first-failed-level-read-undefined
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-rationale-pause-example-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: multi-slot-hold-notices-per-slot
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pool-holders-failure-path-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: gate-hold-keys-unvalidated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: go-timer-clock-fact-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tolerant-records-named-for-yaml-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.

The two BLOCKING keys concern the run watch's own mechanics.
One is where its memory lives against the pass step's cleanup of done runs.
The other is a skip that reads the landing notice's memory after that memory is gone, and that also silences cancelled runs.
The other watch keys concern which readings count as a stop, which cancels the watch can see, and when its step runs.
The gate-hold keys concern notice dedup, a failed holder read, config ranges and an unpinned clock fact.
