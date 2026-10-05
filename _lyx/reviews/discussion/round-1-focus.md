---
round: 1
focus:
  - "decision-record.md D1 and D5: the new BLOCKING edge (a redundant test is a finding). Settle whether the bound, naming the covering test, is stated in every Decision that introduces the edge, and whether any path lets a reviewer block without naming one."
  - "decision-record.md D11: the `t.Parallel` opt-out (process-global state named in a comment). Settle whether what it lets through and what bounds it is stated, and whether a file-level comment can excuse a test that touches no shared state."
  - "decision-record.md D13 against Scope, Constraints and Acceptance criteria: settle whether the already-committed PATTERN half and Plan's remaining half are split the same way in all three sections, with no card left to re-create what D13 says is done."
  - "decision-record.md Acceptance criteria and Scope: settle whether text restating D1, D10, D11 or D12 wording verbatim is needed by the plan writer or only duplicates a Decision. Relocate narrative, but extract any constraint into Decisions or Constraints first."
  - "support-log.md Question ledger and Interview against decision-record.md: settle whether any requirement the plan writer needs lives only in the support log, since Plan-Write never reads it (for example the Q10 reasoning, or a rejected alternative whose constraint is missing from Constraints)."
  - "decision-record.md D2 to D9 against each other: settle whether the phrases to remove (Acceptance criteria) and the phrases to add (D2, D3, D4, D9) are consistent, and whether D9's no-new-test claim holds against the tests the other Decisions imply."
---

The Do-not-flag list rules out three items: a missing Notes subsection (present anyway), missing rejected alternatives in the record, and incomplete call-site enumeration.
The discussion stencil tells the writer not to gather signatures, `file:line` citations or interface shapes, so none of those is a missing-content finding.
