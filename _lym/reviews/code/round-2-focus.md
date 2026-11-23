---
round: 2
focus:
  - "Check internal/boardengine/store.go Store.validateWrite against the decision that a task or a note may depend on any other entry, a note included, with cycles still refused: does any refusal of a note as a dependency target, or of making a depended-on entry a note, remain?"
  - "Check internal/boardengine/dependency.go Dependencies.metBelow: does a task waiting on a note stay pending until the note is promoted and lands or is removed, whatever the note's own status?"
  - "Check internal/raddleparser/check.go and contracts/specs/raddle-proposal-spec.md for any depends-on-a-note refusal that contradicts the decision that a note may be depended on."
  - "Check internal/boardengine/store.go Store.RemoveTask and Store.MergeTasks against the decision that removing an entry strips its slug from every other entry's depends_on, as Prune does: does a removed slug stay anywhere, and does a merge that upserts the slug again keep its dependency?"
  - "Check plugins/ly/skills/board/SKILL.md, internal/boardengine/doc.go and the BriefTask doc comment in internal/boardengine/store.go against the diff's behavior: do they state that a note may be depended on, that removal strips depends_on, and that BriefTask computes Pending at read time?"
  - "Check any test the round adds or changes in internal/boardengine/store_test.go and internal/boardengine/siblings_test.go for duplicated coverage of another row's behavior."
---

Round 1 left scope gaps in note dependencies and entry removal, plus the docs that state them.
