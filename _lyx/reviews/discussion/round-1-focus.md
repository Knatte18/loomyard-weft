---
round: 1
focus:
  - "D7, the vanished-session and absent-engine rules: does each state what it lets through (a silent undercount) and what bounds it, given the Goal says a silent undercount is worse than a refusal?"
  - "D1 and D2, cross-engine union and dedupe: is a session located by two engines, or a message key that collides across engines, bounded by the engine-and-key dedupe rule, or can it double-count or drop a session?"
  - "D4 and D3, unpriced and zero-usage: do the pricing rules agree with the acceptance scenarios (an engine without pricing, an unknown model, a zero-usage message), with no path that prices a nonzero message as free?"
  - "D6, the offline read set: does the record state what a zero-config engine construction can skip or let through, and what keeps it read-only?"
  - "D2 against Acceptance criteria: does every neutral-entry field the acceptance criteria rely on (turn flags, launch-line prompt path, tool-result linkage, repeat count) appear in the D2 field list, with no premise about the transcript that contradicts the Goal?"
  - "Scope against Decisions: does any Decision or Acceptance criterion change observable report output beyond the one named exception, or does Scope name something no Decision covers?"
---

Focus entries name where to look; none pre-states a verdict.
Entries were checked against the rubric's Do-not-flag list and the discussion spec's symmetry rule: none asks for rejected alternatives in `decision-record.md`, a plan-writer notes subsection, or call-site enumeration.
