---
round: 1
focus:
  - "Decision 4 versus Decision 6: a readable transcript with no `skill_listing` attachment, or a truncated listing, is not covered by the unverified degrade. Settle whether Decision 4 then classifies every skill `unknown` (a silent skip of skills that exist) or degrades, and whether a Decision states which."
  - "Decision 7 versus Decision 4 and the Goal: on a load-turn timeout there is no turn end and no transcript read, yet the Decision skips 'every skill not confirmed'. Settle what confirms a skill on that path, and whether the Goal's 'skipped at the end of that turn' still holds."
  - "Decision 11 versus Decision 12 and the orch acceptance criteria: settle whether the persisted state (step, first-typing time, events offset, skills to retry) is enough for a restarted watcher to resume a retry step without re-classifying or sending a second retry, and whether an old `reload_step` read as the pointer step can skip the skills and the retry silently."
  - "Decision 13 rests on re-injection after compaction, which the Auto-mode assumptions mark as not verified. Settle whether the Decision's bound states what a wrong premise lets through, or whether it presents the gap as closed."
  - "Decisions 5 and 6: the bound 'at most two load turns, each bounded by the load timeout' against a pane that dies, a transcript that goes unreadable between the two turns, and a retry turn that names skills the first turn already loaded. Settle whether each case has a stated outcome."
  - "Decision 8 removes the pane-text classifier, a guard against an unknown-skill stall. Settle whether the Decision states what the transcript check lets through that the classifier did not, for example an unknown skill on an engine or turn where the transcript is unreadable."
  - "Decision 14 and the Open risks hand-check bullet: settle whether any requirement in them is missing from the Decisions or Acceptance criteria, and whether any deliberation narrative in them belongs in `support-log.md` instead. Extract a requirement into the record before relocating any prose."
---

Focus set from reading the decision record and support log only.
Candidates were checked against the rubric's `Do not flag` list: none asks for a missing "Notes for the plan writer", rejected alternatives in the record, or call-site enumeration.
None asks for exact signatures, `file:line` citations, interface shapes or dependency lists, which the discussion stencil tells the writer not to gather.
