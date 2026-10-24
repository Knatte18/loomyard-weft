---
round: 1
focus:
  - "internal/boardengine/taskguard.go and board.go: does the net-change check refuse every path that leaves a new task (upsert of a new task, note turned task, promote, upsert-batch, merge) while a rename, a new note and an edit of an existing task pass, and is board.json left unchanged on refusal?"
  - "internal/boardengine/taskguard.go: does the strand-name test exempt the orch role and slug-free names (empty, hub orch, `<shortname>:<slug>:orch`) and refuse the task-slug names of other roles, matching how the run_settings and hub-config guards parse a strand name?"
  - "internal/boardcli/cli.go: does every write verb's board carry the strand name from LYX_STRAND_NAME, read only in boardcli and never in boardengine?"
  - "The refusal message: does it name the strand and the entry and give the `\"kind\":\"note\"` way forward?"
  - "The stencil diffs under contracts/stencils: does each of the Discussion, Plan and Darn writer stencils and the two review rubrics carry the note-never-task rule, without dropping or weakening an existing rule, and is the orch role stencil left unchanged per the withdrawn D5?"
  - "doc.go in boardengine and boardcli: do they describe the new guard, and does the diff add no scope the board entry and record do not license?"
  - "Test economy in taskguard_test.go, verbs_test.go and cli_test.go: does each new test cover behavior no existing test covers, and does any acceptance behavior have no coverage?"
---

Seed focus drawn from the board entry, the decision record with its D5 withdrawal, and the diff stat of the review range.
