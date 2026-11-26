---
round: 2
ledger:
  - key: watch-memory-deleted-for-done-run
    rounds: [1]
    status: resolved
  - key: landed-cancelled-skip-misfires
    rounds: [1]
    status: resolved
  - key: absent-child-part-reads-as-stop
    rounds: [1]
    status: resolved
  - key: record-removing-cancel-unreported
    rounds: [1]
    status: resolved
  - key: stop-change-rule-d2-vs-d3
    rounds: [1]
    status: resolved
  - key: watch-held-with-scheduling-unbounded
    rounds: [1]
    status: resolved
  - key: first-failed-level-read-undefined
    rounds: [1]
    status: resolved
  - key: d1-rationale-pause-example-false
    rounds: [1]
    status: resolved
  - key: multi-slot-hold-notices-per-slot
    rounds: [1]
    status: resolved
  - key: pool-holders-failure-path-missing
    rounds: [1]
    status: resolved
  - key: gate-hold-keys-unvalidated
    rounds: [1]
    status: resolved
  - key: go-timer-clock-fact-unpinned
    rounds: [1]
    status: resolved
  - key: tolerant-records-named-for-yaml-record
    rounds: [1]
    status: resolved
---

Round 1 raised every key above, on the run watch's mechanics and the gate-hold notice.
Round 2 resolved all of them and raised no finding.

The round-2 review lists further defects below its incremental bar.
They are not findings, so they have no ledger key; the round-2 verdict file names the MEDIUM ones.
