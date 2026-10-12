---
round: 1
focus:
  - "Decisions 2, 4 and 7: settle whether the orch row's transcripts enter the idle union and the role-time unions, and whether 'union of working time plus idle is the span' holds once orch spans several runs and is clipped; the record names no rule for orch's intervals."
  - "Decision 1: settle whether the interval rule is fully determined for every first-line kind (user line, assistant line with null or absent `stop_reason`, parse failures, equal timestamps, lines out of order across files), and whether the Auto-mode assumption and the Open risks agree with the Decision's text."
  - "Decisions 3 and 5 with the acceptance criteria: settle whether 'elapsed sums to the span' holds for every case the record admits (history entries out of order or with `last_at` earlier than `at`, a first entry ending before `started_at`, a span start from the seed versus the earliest transcript line), and whether each producer's idle is well defined when intervals are clipped."
  - "Decision 8, attack-surface question: the weft default now applies whenever `-weft` is empty. State what that can now let through or break (a wrong sibling weft, a slug resolved against the wrong repository) and whether the bound is stated in the Decision itself rather than only under Open risks."
  - "Decision 9: settle whether the `shedrun` helper for bytes read from git or a run id without alias resolution is bounded, so it does not become a second decoder or a second declarer of the `shed` segment that PATTERN-shed-run-directory forbids."
  - "Relocation check: look for deliberation narrative inside `decision-record.md` (rationale text that re-argues a choice, or Auto-mode assumptions that restate a Decision) that could move to `support-log.md`; apply the completeness-before-leanness test before raising any, and extract any requirement the plan writer needs first."
---

Seed focus for round 1.
Entries name where to look and the question to settle; none pre-states a verdict.
No entry targets a missing "Notes for the plan writer" subsection, missing rejected alternatives in `decision-record.md`, or call-site enumeration, per the rubric's `Do not flag` list.
The discussion stencil tells the writer not to gather exact signatures, `file:line` citations or interface shapes, so none of the entries asks for them.
