---
round: 2
focus:
  - "In decision-record.md D2, check the failure path of insert-card and remove-card once files are renamed or deleted: does the plan end up consistent, and does each refusal row's stated way forward match the state the failure leaves (the loom raw-I/O row in contracts/specs/refusal-spec.md says nothing is mutated and the verb is re-run, while D2's own precondition refuses a plan whose Card Index and card files disagree)?"
  - "In decision-record.md D2, check that every bound on insert-card's new file write is checkable: the slug rule it cites must exist in contracts/specs/loom-plan-spec.md or be defined in D2 (the parser's cardIndexLineRe in internal/planparser/parse.go accepts any non-whitespace slug, and cardFileName joins it into a file name), and the --intent value must not be able to break the Card Index line the verb writes."
  - "In decision-record.md D2, check whether the begun-card bound is read and applied atomically against webster's begin-batch, which records batches under websterengine.AcquireStateMutation, and whether the begun set is stated as every batch record in the run record rather than only the terminal-done set that loomshed.ValidatePlan reads."
  - "In decision-record.md Open risks and D5, check that each open risk is a real operator tradeoff rather than a question the code already answers (autoRebaseline and Rebaseline in internal/websterengine and internal/webstercli), and that the D5 reachability rule is scoped to refusals and guard branches a card prescribes."
---

Focus targets the failure, bound and atomicity edges of the new rename and delete verbs, plus the open-risk placement.
