---
round: 1
focus:
  - "internal/boardengine/store.go validateWrite and siblings.go: does a task or note depending on any entry kind (task, note, project) pass, and is a cycle through a note refused as for tasks?"
  - "internal/boardengine/store.go and siblings.go: does removing an entry strip it from every other entry's depends_on, as prune does for a done entry, for notes and tasks alike?"
  - "Promote path: is a note's depends_on kept, and does one in the promote payload replace it?"
  - "Task depending on a note: does the board show the task waiting until the note has become a task and landed or is removed? Check the readiness logic against the added decision."
  - "internal/boardengine/render.go noteTable and entrySections, boardcli/text.go: do the README and `list --text` show a note's dependencies as they show a task's? Mutation survivors at render.go:682, 703 and 704 name lines no test asserts."
  - "internal/boardcli/cli.go and doc.go: do upsert and set-deps accept depends_on on a note, and do the payload and help schemas still agree?"
  - "plugins/ly/skills/board and board-policy SKILL.md: do they say a note may carry depends_on, with no leftover statement that only a task does?"
  - "Tests edited at the review base (store_test.go, siblings_test.go, replay_test.go, export_test.go): was any assertion widened or removed without a Loosened-Check trailer?"
---

Seed focus from the decision record's Wanted list and the parent's added decision on what an entry may depend on.
