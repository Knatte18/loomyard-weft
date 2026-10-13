---
round: 1
focus:
  - "decision-record.md D8 (self-exec) and D10 (migration): does each state what the new exec path or the pre-flight rename and `move-window` can skip or let through, and what bounds it, including a failed or racing step?"
  - "decision-record.md D1 and D2: the refused second `overview` strand, the plain `new-window -d` fallback when no shell resolves, and the psmux `-b`/`-a` fallback. Does each fallback leave a stated, bounded outcome, or can it silently break the overview-first, shell-last order?"
  - "decision-record.md D5 and D6: do the focus rules and the one-conditional-step key bindings hold when several hidden or dead-pane windows sit together, or when the viewed window ends during a replace? Look for a false premise, not a missing enumeration."
  - "support-log.md Interview and Question ledger: is any requirement or constraint the plan writer needs stated only there (for example the re-exec'd pane's stale sidecar, which makes the next `lyx loom start` replace it again)? Extract it into decision-record.md's Decisions or Constraints before judging any relocation."
  - "decision-record.md as a whole: does any narrative belong in support-log.md (history, commit-hash asides, deliberation in Scope Out or Rationale lines), or any item in Scope or Auto-mode assumptions repeat a Decision without adding a requirement? Relocation findings count equally with gap findings."
  - "Acceptance criteria against Decisions: does each Decision with observable behavior (D3, D7, D9, D10) have a criterion, and does any criterion assert something no Decision states?"
---

Seed rationale: the record is already lean and carries bounds on D1, D5 and D8.
The risk areas are the fallback paths and migration (D2, D10), the focus and navigation edge cases (D5, D6), and requirement text that lives only in the support log, which Plan-Write never reads.
Not seeded, per the rubric's Do not flag list and the writer stencil: rejected alternatives in decision-record.md, call-site enumeration, exact signatures and `file:line` citations, and the optional plan-writer notes.
