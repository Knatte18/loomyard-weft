---
round: 1
focus:
  - "D1/D3 in decision-record.md: does the known-key rule (encoding/json case-insensitive match) stay consistent with D3's consumption of `tier`/`type` and with `storedEntry` embedding `Task` (Constraints)? Settle whether a plan writer can implement the decode without a hidden conflict, such as a duplicate or case-variant key kept and also decoded."
  - "D4 and the Acceptance criteria: does every write path D4 lists (set-status, upsert, batch upsert, promote, prune, set-deps, merge, remove) have a matching criterion, and do the criteria and D4 agree on merge and remove behavior?"
  - "D4/D6 attack-surface question: does keeping raw unknown values, and not refusing a newer schema, state what can now pass through and what bounds it? Check each Decision's Bound line against what it actually limits."
  - "D7: does the premise that every run-directory-creating process sweeps and many sweepers race hold, and do the time-budget retry, the unexported seam and the Windows Open risk fit together? Settle whether the stated bound (one second, leaf imports) is enough for a plan writer to act on."
  - "Relocation and leanness: is any deliberation narrative in decision-record.md that Plan-Write does not need, and that could move to support-log.md? Apply the completeness-before-leanness test first: extract any requirement into Decisions or Constraints before moving prose."
  - "Scope vs Goal consistency: the Goal says the three remaining PR #574 items are 'confirmed already met'; check that the Already-met list gives the plan writer enough, in the record itself, to know no card is needed, since Plan-Write never reads the support log."
---

Focus covers where design premises, bounds and record/log placement are most likely to be wrong.
The `Do not flag` list was applied: no entry asks for rejected alternatives in the record, a plan-writer notes section, or call-site enumeration.
