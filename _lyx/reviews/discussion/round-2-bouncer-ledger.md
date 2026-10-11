---
round: 2
ledger:
  - key: marker-write-vs-host-write-scope
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-marker-list-misses-host-specific-refs
    rounds: [1]
    status: resolved
  - key: d6-bound-covers-record-only-and-nested-forks
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: relaunched-writer-lost-forks-undefined
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: poll-misreads-daemon-wakeups-and-fswatch
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d9-carries-provenance-and-consumer-list
    rounds: [1]
    status: resolved
  - key: missing-criteria-d3-reusability-d2-write-scope
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings, none BLOCKING; round 2 raised four, none BLOCKING.
Three round 1 keys are resolved: D3's marker list, D9's provenance and caller list, and the two missing criteria.
Four keys persist.
The write-scope key (F1) now concerns the host template's "two files" lead-in, the marker's parent directory and an over-strong acceptance criterion.
The D6 key (F2) now concerns the fork-only bound holding only while `claude_deny_agent_tool` is true.
The relaunched-writer key (F3) now concerns a death before the marker, and what prompts the relaunched writer's next turn.
The poll key (F4) has narrowed to a LOW wording conflict between the support log's rejected alternative and D9.
