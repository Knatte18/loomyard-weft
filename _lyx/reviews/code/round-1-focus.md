---
round: 1
focus:
  - "Part 1 removals (D1, D2, D3): look for leftover references to `parent-review`, `internal/alignment`, `internal/parentreview`, `lyx loom review`, and the retired config keys `alignment`, `alignment_timeout_min`, `skip_alignment`, `parent_review_wait_min` in code, stencils, docs and PATTERN; settle whether reconcile and read-time migration drop and report each retired key."
  - "D8 Bouncer `decision` escalation in the shed engine and Bouncer entry: settle whether the escalation runs before the judgment refusal, soft-cap, circling and auto-accept; whether a settled generation with a CONVERGED verdict and an unsettled decision file is handled as D8 states; whether the held-refusal paths (re-escalate below the hard cap, budget-exempt Halt at or past it) and the accept carry-over into `## Open risks` match the decision record."
  - "Row config keys for the Discussion-Bouncer and Discussion-Burler-Fix: settle whether the recipe pins both keys set together on loom's Discussion rows and neither on any other row, and whether `internal/arbiterstop` hands a `decision` escalation back to the parent."
  - "Lenses (D5, D6) in `internal/lenses/template.yaml`: settle whether `goal-scope` and `board-fit` end in their own lists, stay within the fork's read budget, join `review-discussion` after the existing lenses, and leave `twill-discussion` membership unchanged."
  - "Stencil changes (D7, D9, D10 and the darn writer): read each changed stencil against the record's decisions and settle whether any rule the record did not ask to change is dropped or weakened; check the rubric's `decision` definition and the corrected 'Class never changes' sentence."
  - "Part 3 rename `Merge-In` to `Parent-Align`: settle whether fabric's own merge-in (`MergeIn`, `MergeInAt`, `MergeInOnly`, `MergeInProgress`, `MergeVerbMergeIn`, `lyx fabric merge-in`) is left untouched, whether any loom `Merge-In` reference remains, and whether in-flight runs carry across."
  - "Test economy: for new or changed tests, settle whether any duplicates existing coverage or asserts a stencil's static prose, and whether any added behavior has no coverage."
  - "Docs and PATTERN in the same diff: settle whether `doc.go`, `docs/overview.md` and PATTERN entries are updated for removed and added modules, and that the reportengine read-only parse (D4) keeps counting archived records."
---

Seed focus for round 1, drawn from the decision record's decisions D1–D10. It names where to look, not verdicts.
