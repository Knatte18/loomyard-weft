---
round: 2
focus:
  - "internal/fronttask/hub.go startAction: does a test pin the spawned argument vector for the on_land action (adopt check of the subject), the spawn log required by PATTERN-spawn-observability, and the refusal of an action key outside the closed actions table?"
  - "internal/fronttask/hub.go HubSettleDeps: does a test over a real hub board pin the owner read, so OwnedHere is false when another machine owns the subject and true when this machine does?"
  - "internal/fronttask File and internal/adoptcli/fixloop.go failedEnd: when only the record write fails after the entry was created and released, is the outcome reported as filed rather than not filed, and does a test cover that case?"
  - "internal/loomshed/darnwrite.go darnWrite.Call condition-stop branch: when the stop report is empty, does the halt reason still say how to go forward, and is the case tested?"
  - "internal/adoptcli/green.go resumeHaltedChildren: is the subject filter applied before the child status read, so another subject's read failure cannot enter this subject's notice?"
  - "docs/overview.md dobby and beam entry: does it name internal/fronttask and the start-ahead and on_land behaviour of a front task?"
---

The focus covers the round 1 findings so a fresh reviewer re-checks each fix site against the answer key.
