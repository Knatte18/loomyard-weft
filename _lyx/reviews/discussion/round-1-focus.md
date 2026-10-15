---
round: 1
focus:
  - "D4 (key move) and D2 (lazy load): check that the move cannot let a value cross a layer boundary or reach a key the target's local rule refuses, and that the stated bound holds for the run-settings path as well as reconcile."
  - "D3 and D5 (`deny_python` as a neutral switch for every engine): check what a stale or unmigrated `claude_deny_python` lets through before the first walk, and whether Open risks bounds it."
  - "D5 payload contract: the shell guards match a `\"command\"` string value in the payload. Check whether a payload that does not match leaves a guard silently inert, and whether a Decision states what bounds that."
  - "D9 (empty provider tag mapped to the `claude` namer): check what the mapping lets through for non-Claude strands added outside shuttle, and whether the Decision states the bound."
  - "D8 (scrub union at the shared server spawn): check that the stated claim, a required constructor input makes a forgotten site a compile error, matches the construction-site exception for callers that cannot import `shuttleengines`."
  - "Relocation pass over `decision-record.md`: D10's package-doc list and PATTERN background prose, and any Decision text that narrates deliberation rather than carrying a requirement. Apply the completeness-before-leanness test before proposing a move."
  - "Consistency between Scope, Decisions, Constraints and Acceptance criteria: each Scope item maps to a Decision, and each Acceptance criterion is traceable to one."
---

Seed focus only; the reviewer judges every other part of the pair on its own.
No entry touches the optional Notes subsection, rejected alternatives in the record, or call-site enumeration.
