---
round: 2
focus:
  - "Driverless seed route in D3 and D4: after `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`, does each documented next command (`lyx loom step`, `lyx loom run`) reach a producer without refusing or naming `lyx loom start`, given that `loomPreRun` in `internal/loomcli/arm.go` refuses `run` when no status file exists? Settle whether the Decisions, Scope and acceptance criteria agree on the order."
  - "D1's turn-start rule in `_lyx/discussion/decision-record.md`: does it state what happens to a long gap that follows a non-turn-ending line and ends at an assistant line, a `tool_result` line or an interrupt-marker line, and does its Rationale and Open risks rest on what real transcripts under `~/.claude/projects/` show rather than an assumption? Settle which long waits it keeps as working time and which it gives up."
  - "D1's Bound and compaction: a `/compact` command line is followed minutes later by a string-content user summary line. Does the turn-start definition say whether that interval is working or idle, and does the Bound's claim about mid-turn user lines hold for it?"
  - "D3 and D4's typed parent branch: does a Decision state what a wrong `<parent-branch>` value lets through, given that `seedAndCommitBootstrap` and `shedrun.WriteSeed` refuse a disagreeing seed, and does it name a way forward that clears it or record the stranded case under Open risks?"
  - "D2's transcript filter: is the case of a transcript with no timestamped user or assistant line decided, and does the decision agree with how the session count is filtered?"
---

Round 2 should re-check the same Decisions against the revised record.
