---
round: 1
focus:
  - "internal/gitrepo: check that every go-git read goes through the one retrying helper and that `goGit()` has no other caller; settle whether the retry re-runs the whole read, not just the failing step."
  - "internal/gitrepo `HeadContains`: check the committer-time bound on the walk stops at the plan's slack before the target and does not miss a target on a skewed-clock history."
  - "internal/proc and internal/gitexec: check verifyrun's process-group kill was moved, not copied, and that every remote git command gets the deadline, the kill and the low-speed environment, with a timeout classed as a transport failure."
  - "hub store gc (fabric and fabricengine): check `gc.auto=0` and `maintenance.auto=false` are set on both hub stores and the pair-removal gc is best-effort, so its failure never fails the removal."
  - "batten `--window`: check a done slug is refused before any window opens and that the check is the shared status check, not a duplicate."
  - "Per-card mechanical check: for each card, confirm the type-specific checks its groups name (`Rename`, `Delete` and the like) ran against every target of the group."
  - "Test economy: check each new test against existing coverage, and each added behavior for a covering test; confirm `cmd/lyx/testdata/test-budget.yaml` rows move with the tests that need them."
---

Seed focus drawn from the plan's card index; it sets where to look, not what to conclude.
