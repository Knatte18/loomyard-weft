---
round: 2
focus:
  - "Where a records side returns nil from `PushPairAnchored` without landing (a deferral or an ended branch), settle whether the transition seam can still read it as landed and start the next producer before the transition is on origin."
  - "Wherever a card moves a branch backwards (origin's code branch through `UpdateRemoteBranchLeased`, or a local parent branch through `git reset --keep`), settle whether the step runs through an executor in `internal/fabricengine/destroy.go` that checks containment, ownership, dirtiness and force in that order, as `PATTERN-fabric-boundary` requires."
  - "For every field or signature a card removes or changes, settle whether each test and caller that sets or calls it across the repository is a target of that same card, so the card's own gate compiles."
  - "For each card that reads origin state (batten record, run record, code tip), settle whether the reader it names reads that record from the branch where the record actually lives."
  - "For each card that force-moves, gives up, or discards a tip or transition, settle whether its Intent states what it can drop and what bounds it, and whether a tip held in this machine's own worktree is pushed before it is given up."
---

Focus is on the guard-reach and affected-set questions that produced the BLOCKING findings, restated without reference to the prior review.
