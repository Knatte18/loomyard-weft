---
round: 1
focus:
  - "internal/loomcli/prepush.go: does the check run before every point the board entry names (commit for a push, push, land), and does any push path in loomcli/tendercli bypass it?"
  - "internal/loomcli/prepush.go: when the owner moved to another machine, does the run stop before pushing with a stop that names `lym adopt`?"
  - "internal/loomcli/prepush.go: when only origin's tips moved, does the run take the new base the way a landing does, and stop naming what moved when it cannot? Check failure paths such as a failed fetch, a missing remote or a detached state."
  - "internal/loomcli/prepush.go: does the orch get what moved in one short notice line, and is the compare against what the run last saw (base, parent branch, owner, beam, root) all five, not a subset?"
  - "internal/tendercli/deps.go and internal/loomcli/wiring.go, landingdeps.go, commitrecords.go: do the new dependency seams keep existing callers and behavior unchanged?"
  - "internal/loomcli/prepush_test.go: does each added behavior have coverage, and does any test duplicate an existing test's coverage?"
  - "PATTERN.md and pattern/PATTERN-owner-machine.md: does the pattern change match the code change, and do the doc.go edits state what the code now does?"
---

The decision record carries only the board entry under `## Goal`; the review measures the diff against that entry.
