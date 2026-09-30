---
round: 1
focus:
  - "decision-record.md Constraints (Fabric Destruction Chokepoint bullet) against Acceptance criteria bullet 1 and support-log Question ledger #6: `Add`'s create path forks with `git worktree add -b <weft> <path> <parentWeft>`, which checks the parent's `_lyx/shed/` out onto disk immediately. Settle whether an index-only removal really 'avoids the question' when the criterion demands 'no such files on disk', and whether leaving the mechanism open for Plan leaves the record internally consistent."
  - "Decision 2 (adopt path drops nothing): settle whether 'an existing branch is the pair's own' holds for every way `Add` reaches `weftBranchAlreadyExists`, e.g. a retried `Worktree-Create` or `lyx fabric add` after a crash between the fork and the drop commit, or a leftover weft branch, which could carry the parent's run records into an adopted pair."
  - "Decisions 5 and 7, and Acceptance criteria bullets 1-2: check that the single-commit rule (drop only when the fork point tracks something under the root; commit message naming the origin record or both) and the lock-wait verdicts (Stuck at the bound, cancel error, hard error on Acquire failure) are stated precisely enough for Plan to test against, with no contradiction between the Decision text and the criteria."
  - "Leanness of decision-record.md: Decision 4's import-list rationale, Decision 7's logging/seam details, and the Auto-mode assumption about pairs the hub cleared by hand. The writer stencil says dependency lists and similar Plan-time facts are not to be gathered. For each, settle whether it carries a requirement Plan needs (keep it) or is deliberation to relocate to support-log.md or drop, applying the completeness-before-leanness test."
---

Seed rationale.

The board body explicitly folds `batten-prime-lock-wait` into this task and suggests a batten namespace plus a no-inherit create, so the scope absorption and the namespace supersession are grounded in the brief and are not focus entries.

The first entry comes from `internal/fabricengine/add.go`: the create path checks out the forked tree before the drop, so the decision record's constraint and its first acceptance criterion may pull in different directions.

The Notes for the plan writer file list is an optional head start and is not a focus.
Neither is call-site enumeration, per the rubric's Do-not-flag list.
