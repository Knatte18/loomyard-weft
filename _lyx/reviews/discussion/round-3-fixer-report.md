# Round 3 fixer report — Discussion-Review

All three findings were validated against the code and the record and fixed.
`lyx loom validate-discussion` exits 0 after the fixes.

## Fixed

- **F1 (marker recognition).**
  Valid: D2 never said what makes a line a marker line, and D2's own inline spelling of the marker would trip a whole-file substring scan.
  D2 now defines a marker line as a line equal to the opening or closing marker text after trailing whitespace is trimmed, counted only inside `## Open risks` and outside fenced blocks; marker text elsewhere is prose.
  The refusal now covers every malformed marker set: unclosed opening line, unopened closing line, two pairs for one segment, a closing line naming another segment, a nested pair.
  Acceptance criteria: the unclosed-marker criterion is widened to the full malformed set, and a new criterion pins a record that spells the marker text inline, in another section or in a fence still settling with that text unchanged.
- **F2 (bound cannot fail).**
  Valid: comparing the bytes before and after the entry with the prior file they were cut from holds by construction.
  D2's Bound now states that the splice keeps the other bytes by construction, so marker recognition and the one-line rendering are what bound the write.
  The check that can fail is a re-read of the result before the file is replaced: same H2 heading sequence as the prior file, exactly one well-formed pair for the segment (none after a removal), and every other segment's entry byte-identical.
  The refusal sentence names a failed result check instead of a failed byte check.
  The support log's Rejected alternatives records the prefix/suffix comparison as rejected, and why.
- **F3 (Goal and Open risks claim a guard).**
  Valid: no Webster stencil reads the decision record (only `webster-prefix-recovery.md` mentions the plan's decisions), and `loom-rubric-plan-review.md:63-64` licenses card scope from the record in a rework generation.
  The Goal now names each segment's consumers: Discussion-Review's entries go to Plan-Write and rework, while Plan-Review's and Webster-Review's go to Describe.
  The first Open risks bullet now states that a wrong Plan-Review fix is first checked at Describe, after Webster has built it, as the accepted bound, with PR-Gate's rejection into rework as the way back.
  D2 now adds one sentence to the Plan-Review rubric's answer-key bullet: a carry-over entry is not part of the answer key and licenses no card's scope.
  The Webster-Review rubric stays unchanged, because it holds the code against the plan.
  Scope lists the rubric change, an acceptance criterion pins it, and the support log records why the rubric does not stay unchanged.

## Disputed

None.

## Deferred

None.
