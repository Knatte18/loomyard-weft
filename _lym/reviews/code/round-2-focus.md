---
round: 2
focus:
  - "internal/shedarbiter/edits.go readEdits and the WorktreeSeam implementation in internal/loomcli/arbiter.go: does an untracked file the arbiter leaves in the task's code checkout, committed or not, get caught before the run resumes? Settle whether the spawn snapshot and the post-arbiter snapshot both cover untracked paths."
  - "internal/shedarbiter/call_test.go TestCall_Edits and its fakeWorktree: does a row reach each failure of readEdits after the arbiter, namely the snapshot read, IsAncestor and CommitLog, separately from the spawn-time read? Settle whether each expected hand-on reason names the failure and that HEAD moved."
  - "contracts/stencils/loom/loom-template-arbiter.md and loom-fragment-arbiter-fix.md: in fix mode, does the text still forbid every git command that changes the repository other than add and commit on the found branch?"
---

The focus follows the round's gating findings and the fixes they call for.
