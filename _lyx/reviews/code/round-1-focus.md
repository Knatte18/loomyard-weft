---
round: 1
focus:
  - "internal/boardengine/taskguard.go and the board.go write critical section: does the net-change check refuse every task creation path (new task, promote, note upserted to kind task, merge upserting a new task, upsert-batch) and let a rename, a new note, a status or run_settings write and an edit of an existing task pass?"
  - "internal/boardengine/taskguard.go: does the orch-role exemption cover the hub orch, slug-free names and numbered orch threads (`<shortname>:<slug>:orch` and numbered variants), and is a task session's strand name parsed the same way the existing run_settings and hub-config guards parse it?"
  - "internal/boardcli/cli.go: does every write verb's board carry LYX_STRAND_NAME, and is the strand name read only in boardcli, never in boardengine?"
  - "Amendment scope: the parent's changes to boardcli/cli.go, doc.go, verbs_test.go, boardengine/doc.go and the board SKILL.md since c9c4e7d; is the task's guard still wired and documented consistently after the merge?"
  - "Stencils (darn-template-write, loom-template-discussion, loom-template-plan, discussion and plan review rubrics): do they say a follow-up becomes a note, never a task, without dropping or weakening an existing rule, and is the orch role stencil unchanged, since D5 is withdrawn?"
  - "Docs in the same diff: do boardengine/doc.go, boardcli/doc.go and SKILL.md state the refusal, the exemption and the stated bound accurately against the code?"
  - "taskguard_test.go and verbs_test.go: does each acceptance criterion have coverage, without duplicating an existing test?"
---

Set from the board entry, the decision record and the amendment overlap note.
