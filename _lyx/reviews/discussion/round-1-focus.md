---
round: 1
focus:
  - "decision-record.md D1 and D2, the stub claim: does the plan gate really report a heading-only card file as missing fields, so a forgotten stub cannot pass? Settle it against the plan spec's validation checks."
  - "decision-record.md D2, the begun-card bound: does reading the run record the way `loomshed.ValidatePlan` does yield the begun set, not only the done set, and does refusing positions at or before the last begun card leave no way for a renumber to desynchronize webster's run record from the unbegun cards that move?"
  - "decision-record.md D2, the apply order and partial failure: do the stated move orders (descending for insert, ascending for remove, Card Index last) actually avoid two files claiming one number at every step, and is the partial-state outcome bounded as the attack-surface question requires?"
  - "decision-record.md D2, `remove-card` and `insert-card` bounds: does each refusal in the list match an Acceptance criteria line, and is every new edge (file rename, file delete) bounded by a stated limit and not only by the happy-path description?"
  - "decision-record.md D5 against `PATTERN-refusal-way-forward` and `refusal-spec.md`'s wiring-guard class: is the claim that a fail-loud wiring guard takes no per-row test a true reading of those rules, or a false premise?"
  - "decision-record.md D4: is the claim that the Plan-Review rubric reaches the Plan-Burler fixer through its round prompt, without touching the burler stencils, a true premise, and does the rubric's `## Write surface` section exist as described?"
  - "decision-record.md Open risks and Acceptance criteria: is the unresolved `lyx webster rebaseline` question a design gap the plan writer cannot settle alone, or a legitimately deferred plan-time check; and does any content in Open risks or Notes for the plan writer belong in support-log.md instead (apply the completeness-before-leanness test before moving anything)?"
---

Seed rationale: the record is lean and its rejected alternatives sit in the support log, so the focus is on premises the design rests on (gate behavior, run-record reading, rubric delivery, wiring-guard rule) and on whether each new rename or delete edge is bounded.
No entry asks for rejected alternatives, a Notes subsection or exhaustive call-site enumeration.
