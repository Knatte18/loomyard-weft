---
round: 1
ledger:
  - key: driverless-seed-run-still-refused
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: abandoned-turn-gap-not-caught
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: compaction-becomes-idle
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: placeholder-parent-branch-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-no-timestamp-transcript-undecided
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: rejected-alternative-in-decisions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, all open.
The driverless seed route fails for `lyx loom run` because `run` needs a status file that only `start` or `step` creates.
D1's turn-start rule misses long stalls ending at assistant or `tool_result` lines, and it turns compaction time into idle time.
The placeholder parent branch has no stated bound, and a wrong value strands the agent.
D2 leaves the no-timestamp transcript undecided, and rejected-alternative argument sits in D4 and D5.
