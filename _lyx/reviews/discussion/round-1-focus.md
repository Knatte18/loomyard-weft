---
round: 1
focus:
  - "D3, repeat check: settle whether `findings equal the last routed conflict` is a stable comparison, i.e. whether the findings' rendered text (line numbers, card ids, ordering) can differ between two runs of one unchanged defect, so the check either never fires or fires wrongly; and whether the Decision states what the comparison is over."
  - "D3 premise: check against `BurlerReviewProducer.Call` in `internal/shedadapters/burlerreview.go` that a reroute into `Plan-Burler-Review` after an approved episode really spawns a reviewing round as round N+1; a false premise here is a `design` finding."
  - "D3 and D2, record lifetime: settle whether a conflict record can exist on a `websterOutcomeStuck` that came from a cause other than the begin-batch refusal in this same Webster call (the master stuck for another reason after an earlier refusal in the call), and whether the Decision bounds that."
  - "D3, bound on the new routing edge: settle whether the stated bounds (differing findings, shed reroute cap, `done-card-edited`, `ErrRebaselineCardSetChanged`) cover a reroute whose plan edit touches only pending cards but still leaves the dispatch finding in place, and whether any guard is bypassed by rerouting every blocking finding class rather than `delete-before-reference` alone."
  - "D1: settle whether the `plan_defect` / `tree_moved` split on `DispatchScope`'s begun set is stated precisely enough that the plan writer cannot misread it, including the re-begun first batch case and the operator-commit case listed in Open risks."
  - "D6 and D7, attack surface: settle whether the stdin-only terminal test and the whitespace-only abort state what they let through (a caller with a terminal but redirected stdout, a layer that can no longer be emptied through the editor) and what bounds it."
  - "Relocation check: look for deliberation narrative in `decision-record.md` Decisions or Open risks that belongs in `support-log.md`, applying the completeness-before-leanness test first so no requirement the plan writer needs is lost."
---

Seeded from a read of both artifacts and `loom-discussion-spec.md`.
No entry asks for rejected alternatives, a plan-writer notes subsection or call-site enumeration, all of which the rubric excludes.
