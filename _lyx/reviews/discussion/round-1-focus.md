---
round: 1
focus:
  - "D2, D14, D15: for the binary-change retirement and the archive copy and prune, does each Decision state what it can now skip or let through and what bounds it, including what the prune can delete and what a failed respawn leaves behind?"
  - "D4, D5, D6, D7, D8, D13: each lowers a log level or discards output. Does each state what diagnostic signal is now lost and what bounds the loss, and do the stated bounds hold against the acceptance criteria?"
  - "D9 and D16: does D9 keep draft text out of every orch log line and `State.Stuck` path it names, and does D16's no-retry notice leave the orch a defined way to learn of a landing it never heard about?"
  - "Decisions against Acceptance criteria and Constraints: does each Decision with an observable behavior have a matching criterion, and does any criterion contradict a Decision (D3 episode wording, D8 episode key, D14 collision and partial-entry handling, D15 prune rules)?"
  - "Relocation and exclusion: does `decision-record.md` carry deliberation narrative, forensic detail or tallies that belong in `support-log.md` or nowhere? Check requirement content is extracted into Decisions or Constraints before any move is proposed."
  - "Scope against Decisions: do the Scope bullets and their D-references agree with the Decisions' content, and does anything in Out leak back into a Decision?"
---

Seeded from a read of both artifacts and the discussion stencil.
No entry flags a missing optional subsection, missing rejected alternatives in the record, or incomplete call-site enumeration.
The stencil bars `file:line` and signature gathering, so the Notes for the plan writer pointers are not a gap to fill.
