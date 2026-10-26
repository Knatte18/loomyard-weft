---
round: 1
focus:
  - "internal/sheddriver/driverules.go and drive.go: does the new JudgesFirst row sit after the hand-over and tackler-resume-returned rows and after landing-halt, budget-exhausted, precondition-unmet and pause, and does a nil seam or nil JudgesFirst leave the tackler path unchanged?"
  - "internal/loomcli/drivewiring.go: is JudgesFirst set only for the Webster row, with darn and batten left nil, and does the row name enter only through the seam so the rule table stays recipe-blind?"
  - "internal/discussionparser/append.go and identity.go: does the added-decision read return heading and body verbatim up to the next heading, in record order, skip fenced headings, and stay the only parser of the decision record?"
  - "internal/websterengine/render.go, geometry.go, runlevel.go: is parent_decisions a required marker filled from Geometry.DecisionRecordPath, rendering `none` for a record without entries and for an empty record path, and are failure paths of the record read handled?"
  - "contracts/stencils/webster/webster-template-master.md: does every stuck rung end in `outcome: stuck` alone with no message to the parent, is the stuck-reaches-the-arbiter rule stated once, are the waiting and decision rungs still allowed to message, and does the parent_decisions text tell Merriam to hand answers to forks and not re-ask?"
  - "Docs scope: the decision record lists the orch role stencil where it states the escalation path as a doc to update; does the diff update it or does it need no change, alongside the updated websterengine and sheddriver doc.go?"
  - "Tests: are new behaviours pinned as rows in existing tests per PATTERN-test-speed, and does any new test duplicate an existing one or assert stencil static prose?"
  - "Commit messages and summary: any Pattern-Departure trailer valid, and any claim in _lyx/landing/summary.md the diff does not bear out?"
---

Seed focus from the board entry and decision record, checked against the 6865efba..HEAD diff stat.
The diff stat shows no orch role stencil change, so the docs entry is a question to settle.
