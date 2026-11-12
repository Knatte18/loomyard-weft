---
round: 2
exclude_lenses: []
focus:
  - "D9 in _lym/discussion/decision-record.md opens lym gate testmap to task worktrees: settle how a task worktree's upkeep pass registers in internal/runproc, whether it can block or be refused by the prime dobby's pass (internal/gatecli/testmap.go registerUpkeepPass, internal/dobbyengine/testmap.go TestMapRunning), and whether pairteardown's runproc.End ends it, per PATTERN-child-processes."
  - "D11 imports old bolt maps into the prime's _lym/testcover/: settle whether a call killed partway can lose maps that were never committed, given D5's exemption keeps the leftover set from making the worktree dirty and D4 commits it on the next call, and what bound the record states."
  - "D6 claims two machines read each other's maps as current: settle whether the hub env digest (envconfig stacked vars, machine layer included) and a map commit absent from a non-owner's code store (testcover commitMiss) are stated, and whether the record says both degrade to whole-package runs."
  - "D7 leaves an unchanged map at its older commit and moves the planned-at-HEAD meaning to the exit record's planned_head: settle every reader that compares a map's commit with HEAD (internal/gatecli/testmapaudit.go candidate filter) and whether an acceptance criterion covers it."
  - "D6's owner refusal, D11's import and removal, and D10's default-off upkeep: settle their order, who makes the first prime call after the landing, and what becomes of a non-owner machine's own bolt maps."
---

Round 1 findings in the D4, D8 and D10 wording and the D7 evidence are local corrections; round 2 checks they were applied without new gaps.
The extra-branch departure goes to the parent and is not a focus item.
