---
round: 2
focus:
  - "In decision-record.md, check every Decision that deletes or overwrites a path named by an operator flag (the work directory, the output report): does it state what the deletion can reach and what bounds it, including a flag naming the repo root, the hub, a non-empty unrelated directory, or a directory holding the output report?"
  - "In decision-record.md Constraints, check whether any literal the script spells (`_lyx`, `.lyx`) departs from an invariant in PATTERN.md; if so, does the record plan the PATTERN.md change in the same task and list it in Scope?"
  - "In decision-record.md, check every Decision that lets a value read from a log, transcript or flag become a report arm, table cell or path component: does it state the allowed character set, how it is kept from colliding with a classifier-assigned arm name, and what happens to a value outside the bound?"
  - "In decision-record.md, check the rule that resolves a fork's batch from records history: is the search restricted to the fork's own run, so that concurrently running pairs whose batches share a spawn time cannot resolve to each other's cards and start commit?"
  - "In decision-record.md, check the run-name rules for the runs flag: is the match against a project directory exact or by prefix, and does the allowed character set survive Claude Code's directory-name encoding and exclude `.` and `..`?"
---

The questions above follow the rubric's attack-surface question; none targets content on the rubric's `Do not flag` list.
