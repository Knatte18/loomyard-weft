---
round: 3
ledger:
  - key: marker-write-vs-host-write-scope
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-marker-list-misses-host-specific-refs
    rounds: [1]
    status: resolved
  - key: d6-bound-covers-record-only-and-nested-forks
    rounds: [1, 2]
    status: resolved
  - key: relaunched-writer-lost-forks-undefined
    rounds: [1, 2]
    status: resolved
  - key: poll-misreads-daemon-wakeups-and-fswatch
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
  - key: d9-carries-provenance-and-consumer-list
    rounds: [1]
    status: resolved
  - key: missing-criteria-d3-reusability-d2-write-scope
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings, round 2 four, round 3 two; none was BLOCKING.
Findings fell from MEDIUM-heavy to one MEDIUM and one LOW.
The D6 fork-edge bound and the relaunched-writer rule were examined in round 3 and cleared.
The write-scope key persists as a MEDIUM `consistency` wording mismatch between D2 and its acceptance criterion about which host sentences widen.
The poll key persists as a LOW `design` premise problem: D9 and the support log claim the daemon-wakeups rule does not bind a capped call, but the rule's one-minute exemption leaves a 250 s cap unaddressed.
