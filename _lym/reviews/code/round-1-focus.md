---
round: 1
focus:
  - "internal/beamcli/pr.go `prOpen` and `openRecorded`: the answer key refuses opening while the beam conflicts with its parent or a member is still landing. Does every path that makes a pull request ready or reopens it pass `judgeOpenable`, including the open-recorded and already-open cases, or can a dirty beam still reach a ready pull request?"
  - "internal/beamcli/pr.go `judgeOpenable` and `openMergeTask`: does `openMergeTask` match exactly the merge-in member that is still landing for this slug, with no false refusal from another beam's task or a finished one?"
  - "internal/fabricengine/parenttips.go `ConflictsWith`: is `code.MergeTree` clean/conflict semantics read the right way round, and is a merge tree error other than a conflict kept apart from a conflict result? Is the pair's code HEAD the beam's head?"
  - "internal/beamcli/deps.go `beamParentTips` refactor of `ParentUnmerged`: does `ParentUnmerged` behave as before in order of reads, fetch condition and error returns, and is `ParentConflicts` wired in every `landDeps` construction, test fakes included, so no nil call panics?"
  - "Test coverage: does `TestPROpen` cover the conflict refusal, the open merge task refusal and the closed-reopen path? Is the `ConflictsWith` behavior covered in `fabricengine`, or only through a fake seam? Check the `TestHubLandDeps_ReadyReads` change for what it pins."
  - "contracts/orchrole/role-beams.md and internal/beamcli/doc.go: do the doc changes match the behavior the diff lands and the repository's docs rules for a change to a verb?"
---

The measuring stick is the board entry under `## Goal`; the record's other sections are empty.
The commit message claims the check runs before a push or a reopen: verify that against the diff.
