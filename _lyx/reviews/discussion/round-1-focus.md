---
round: 1
focus:
  - "D7, D8, D9 in decision-record.md: for each refusal path, does the Decision state what it can now let through (D8's skip on an unparseable previous ledger, D7's unchanged handling of a verdict already on disk) and what bounds it, and does the bound hold against the D9 re-judge route?"
  - "D5's premise that no Bouncer row in any embedded recipe sets `model`, `effort` or `version`: check it against the embedded recipes, since a false premise makes the retirement a breaking change and a `design` finding."
  - "D2 and D4 against the Acceptance criteria: does every seed and judge resolution case (run-wide `seed`, per-segment seed, per-segment judge fallback) have a criterion that agrees with D2's order, and do any Decisions or assumptions contradict each other?"
  - "D10 and D11: is the single-writer premise behind read-and-replace of `round-<N>-usage.yaml` true for the Bouncer and Burler rows of one segment, and do the malformed-record, failed-write and re-judged-round cases each state one outcome without contradiction?"
  - "Decision numbering and cross-references in decision-record.md (D1, D2, then D4; A1, A3 onward): does any section cite a Decision or assumption that does not exist, or rest on one that the record dropped?"
  - "Relocation check: does any Decisions rationale or Open risks entry carry deliberation narrative that belongs in support-log.md, after confirming the requirement it holds is already extracted into a Decision or Constraint?"
---

Seed focus only; the reviewer still judges the whole pair against the rubric.
Entries avoid the rubric's `Do not flag` items: no entry asks for rejected alternatives, a plan-writer notes subsection, call-site enumeration or exact signatures, which the writer stencil tells the writer not to gather.
