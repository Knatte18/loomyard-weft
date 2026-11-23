---
round: 1
focus:
  - "internal/driftengine/verdict.go `renderVerdict` and `scopeText`: does each repository-facing line the block renders now avoid lym run-model wording (task, branch point, run), or does one remain that the board entry's expectation covers?"
  - "Other renderers of the same block (grep for `Regression check`, `Baseline:`, `regression-checked` outside `verdict.go`, including the `internal/driftengine` check, amendment, range and metadata line helpers): does any still say 'the task' in text a PR reader sees, so the fix is incomplete?"
  - "The `Root-Cause:` trailer on the one commit: does it name the defect the diff removes, the hard-coded run wording in `renderVerdict` and `scopeText`, and not a symptom?"
  - "`internal/driftengine/verdict_test.go` `TestVerdict`: do the two edited expectations still check the cases their names state, with no assertion widened or dropped?"
  - "`_lym/landing/summary.md`: does each claim, including its `## Merge danger` call, match a diff that only changes rendered strings and their test expectations?"
---

The diff is two string changes in `verdict.go` and the matching expectations in `verdict_test.go`.
The board entry asks for neutral wording such as "the base of this branch" in the block.
