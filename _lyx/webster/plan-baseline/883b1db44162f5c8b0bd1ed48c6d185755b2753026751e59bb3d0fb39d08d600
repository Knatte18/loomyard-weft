# Card 1 — Pin the relevant list in the discussion spec

**Prosa:**
- `contracts/specs/loom-discussion-spec.md`

**Intent:**
Add a section for `relevant.md`, the third `_lyx/discussion/` file, per D2 and D4, and reword the plan-writer wording per D4 and D6.
The title names the relevant list beside the decision record and the support log.

The new section pins the format:
a markdown bullet list, optionally under one H1 title;
each top-level bullet opens with one backticked worktree-relative file path, followed by backticked member names in that file in plain Go spelling, unqualified (`PlanTable`, `Config.Validate`), wherever the writer read specific members, then ` — ` and one line saying why it is relevant.
It forbids glyphs, signatures, `file:line` citations and code excerpts, and nested bullets or any other non-blank line.
It states that loom's Discussion-Write lead alone writes the file, and that a darn writer does not;
that the writer records only what it already read and never searches further to fill the list;
and that the list is a head start for the plan writer, never a completeness claim.
It states the list of code the writer read belongs here, not in the decision record.

The decision record's opening line, "the plan writer's sole input", becomes: the decision record is the plan writer's design input, `relevant.md` its code head start, and the plan writer reads nothing else from the discussion directory.
The `## Notes for the plan writer` item is reworded to design hints for the plan writer, never a completeness requirement;
the head-start wording moves out of it into the new section only, and a hint may still name the symbol it is about.
The support-log section keeps saying the plan writer never reads it.

**Commit:** 1: discussion spec: pin relevant.md and the plan writer's inputs
