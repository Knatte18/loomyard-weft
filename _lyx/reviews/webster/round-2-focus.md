---
round: 2
focus:
  - "Comment line breaks: CONSTRAINTS.md's Comment Line-Break Convention, through the scribe:prose rule it cites, allows a break inside a sentence only at an independent-clause boundary, never at a comma that joins a list item or a compound predicate. Settle whether each of these new doc comments now holds one sentence per line: internal/loomcli/parity_test.go around line 546 (the 'The cases are ...' list of rework gate cases), internal/loomcli/wiring_commitstatus_test.go around line 411 (TestStatusCommitPathspec_PendingRejectionHoldsTheRound's doc comment), internal/loomcli/wiring_rework_integration_test.go around line 217 (TestWire_Real_PlanReviewSkipFollowsGenerationClass's doc comment) and internal/loomshed/gates_test.go around line 437 (seedReworkGlyphPlan's doc comment). Also settle whether any other new or changed Go comment in the range breaks at such a comma."
---

Round 1's ten findings are fixed and verified; the only open item is the comment-convention residue above.
