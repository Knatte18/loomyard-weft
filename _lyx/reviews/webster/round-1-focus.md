---
round: 1
focus:
  - "Cards 1-2: does `delete-before-reference` now match a deleted member only through `memberReferences`, scan-only at dispatch and severity-aware in Webster's batch recording and recover-batch count, and is it limited to Go code?"
  - "Cards 3-5: does every plan-file read or write in the check path go through `Plan.ReadFile`, `Plan.RewriteRefs` and `Plan.Reparse`, so `validate-plan --overlay` never touches disk and no path still reads the plan directly?"
  - "Card 5: does `--overlay` handle stdin edge cases (empty input, malformed input, a file that is not in the plan) with a clear error rather than a silent pass?"
  - "Card 6: does the Plan-Review rubric text match the shipped `--overlay` behavior and the enclosing-function naming rule?"
  - "Test economy: do the new tests duplicate coverage of the public surface, and does each added behavior have a test?"
---

Round 1 seed from the plan's card index and shared decisions.
