---
round: 1
focus:
  - "decision-record.md D5 and D6: does the handoff marker have a stated bound against staleness?
    Check whether a marker left by a prior attempt, a retry, or a resumed run (BurlerProducer's retry and partial-file archive) could release the fixer before the current reviewer's checks passed, and whether D5's bound ('only Go writes it, only after D3's checks') covers that case."
  - "decision-record.md Acceptance criteria 'A resumed run whose round has a live half reattaches to it': no Decision states the resume behavior.
    Settle whether a Decision must say what happens when only one half is live or when the marker already exists, or whether D6 plus the criterion suffice for the plan writer."
  - "decision-record.md D8 against Acceptance criteria: the 'Disputed' section must be accepted by the round and not counted as unfixed.
    Settle whether D8 states what the round's own checks do with a disputed finding id, and whether 'the existing deferred section keeps its meaning' is consistent with that, without adding any enumeration of call sites."
  - "decision-record.md D4 and D7: the gate split (review-parse on the reviewer, caller gates on the fixer) against 'verdict and findings parsed only after both runs reached done and the fixer's gate passed'.
    Settle whether the ordering and the failure outcome are consistent across D3, D4, D6 and D7, including which outcome a round returns when the reviewer's gate is exhausted while the fixer waits."
  - "decision-record.md D9 and D10: the scalar-or-list parsing and the removal of row-level model keys.
    Settle whether any claim is a false premise, such as a consumer of the scalar `review` key or of `review_timeout_min` that D9's 'loads unchanged' does not hold for, and whether every Decision's behavior has a matching acceptance criterion."
  - "support-log.md and decision-record.md: relocation check.
    Settle whether any deliberation narrative sits in decision-record.md (Rationale, Auto-mode assumptions, Open risks) that belongs in support-log.md, extracting any requirement the plan writer needs into Decisions or Constraints first; and whether any requirement lives only in support-log.md, which Plan-Write never reads."
---

Focus entries name where to look.
Missing rejected alternatives, a missing optional plan-writer subsection and exhaustive call-site enumeration are excluded by the rubric's `Do not flag` list; the writer stencil also forbids gathering exact signatures and `file:line` citations, so none are raised here.
