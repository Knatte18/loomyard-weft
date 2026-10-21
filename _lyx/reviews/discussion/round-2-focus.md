---
round: 2
focus:
  - "Reconcile's stale-junction removal in `internal/fabricengine/reconcile.go` (`applyStaleRemoval`, `scanOnDiskJunctionNames`, `linkIsFabricOwned`): does the decision record state how the root `.scratch` link survives it on every reconcile, and does an acceptance criterion pin a second reconcile on a converged pair keeping the link and its exclude line?"
  - "Exclude writing for `/.scratch` (`ExcludeAnchoredDir` in `internal/fabricengine/gitexclude.go`): with a tracked `.gitignore` line `**/.scratch/`, does the recorded probe or unconditional write still put `/.scratch` in the exclude so the link is not untracked dirt?"
  - "Step ordering and failure bounds in the scratch step: when the exclude write fails, or the link is refused after a successful rename, does the record state what the worktree looks like afterwards, and do the bound sentence and acceptance criteria match every such case?"
  - "Reconcile entry paths in `repairPairWiring` and `WireJunctionsWith`: does the record say unambiguously where the scratch step runs once per pass on each branch, including when the junction repair fails or the weft was recreated in the same pass?"
  - "Decisions that name a mutation record or another board entry: does each name the concrete mutation kind or the entry it writes, and do the acceptance criteria detect every mutation and the unwire behavior the Decisions state?"
---

Entries name places to look and questions to settle; none pre-states a verdict.
