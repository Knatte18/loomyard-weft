---
round: 1
focus:
  - "D6 (reset verb) and D5 (contract-file evidence rule): attack-surface question. For each, does the Decision state what it can now skip or let through and what bounds it, and does the stated bound hold against the rest of the record, for example whether the 'run's own paths' set and the 'Master wrote later' timestamp rule can be satisfied by something other than the evidence they claim?"
  - "D1 and D3: do the bounds cover the escape hatches they introduce, namely the caller-declared awaited prefixes, the expiry that lets a turn end through, and the stdin-rewriting hook? Check D3 against its own fallback in R1: does the fallback Decision carry a bound, or is a design choice left open on Plan-time verification?"
  - "Consistency between D6's 'pre-fix' target and A7 / the Notes: D6 says the gate's per-shuttle-run attempt semantics stay, A7 says the head is taken at the first failed evaluation. Do these state one rule, and is the persisted value a recorded commit in every case the acceptance criteria exercise?"
  - "Consistency between Scope, D7 and D8: D7's row list and 'at least' wording versus the Scope line 'every refusal ... names `lyx webster reset`'. Is the requirement that the plan writer needs stated as a rule, with the enumeration of rows and Go files a relocation candidate (compiler and Plan-Write quarry lookups own enumeration)? Apply the completeness-before-leanness test before raising it."
  - "D8 leaves the exact command sequence for a shed-driven run to the Plan producer. Is the requirement the plan writer needs (shape, ordering, no restated step) fully in the Decision, or is a design decision being deferred?"
  - "D10: is the closed denied-command list and the scanned surface a bounded rule, and does the Constraints / Acceptance criteria section agree with the Decision on what is scanned and when the list changes?"
  - "Placement: does any deliberation narrative in decision-record.md (Rationale text on why Master reached for a script, the untraceable transcript in D3) carry no requirement the plan writer needs and belong in support-log.md? Does support-log.md Question ledger Q16 match what the record states?"
---

Seed focus only; it names where to look and the question to settle, and does not pre-state a verdict.
Excluded after checking the rubric's `Do not flag` list and the writer stencil: the optional notes subsection, rejected alternatives in the record, call-site enumeration, and exact signatures or `file:line` facts, which the writer was told not to gather.
