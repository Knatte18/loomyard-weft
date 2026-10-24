---
round: 1
focus:
  - "Decision 5 (`inert_paths` and the inert carry) and Decision 8 (`carry: false`): do the stated bounds cover what the empty scope and the ancestor carry can now skip, across all three scopers and at whole gates, and does each Decision state what bounds it?"
  - "Decision 6 (build step) against Decision 3 and Decision 5: do the conditions under which the build runs, is skipped or fails remaining tiers agree across the three Decisions and the acceptance criteria, including the empty-seed and inert-only cases?"
  - "Auto-mode assumptions and Open risks: does any entry leave a fact the design rests on, such as VSTest exit codes for a gated project or MTP behaviour on an unmatched filter, to be found out later, rather than researched with its evidence recorded?"
  - "Constraints against Decision 10: the green-record rule (a filtered or fewer-project pass writes no entry) and Decision 10's claim that Finalize carries Merge-In's whole run; do they hold together, and does the PATTERN note in the last Constraint belong in `## PATTERN changes` or stay as written?"
  - "Leanness: does `## Scope`'s Documentation bullet or Decision 9's repo-rule text carry detail that belongs in `docs/dotnet-hub.md` or the support log, after checking that no requirement the plan writer needs is lost by the move?"
---

Seed pass only.
Candidate points checked against the `Do not flag` list: none relies on a missing plan-writer subsection, a rejected alternative absent from the decision record, or a call-site enumeration.
The record edits no board entry of this task.
