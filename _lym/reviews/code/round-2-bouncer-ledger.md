---
round: 2
ledger:
  - key: done-projects-offered-in-dropdown
    rounds: [1]
    status: resolved
  - key: done-project-nonroot-parent-untested
    rounds: [1]
    status: resolved
  - key: list-text-unchanged-goal-narrowed
    rounds: [1]
    status: resolved
---

Round 1 raised three MEDIUM findings.
Round 2 resolved all of them:
the dropdown skips done projects, the fixture nests done project P5 under done project P3 with an asserted parent,
and the `list --text` finding was settled by an added entry keeping that listing as it is.
Round 2 raised nothing new.
