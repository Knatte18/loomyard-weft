---
round: 1
focus:
  - "internal/fabricengine/onerepository.go `layoutAnswers`: does every path that changes a worktree's layout or its `.lym-anchor` marker call `forgetLayouts`, or can a kept answer go stale? Check marker commits, merges, resets and checkouts outside `commitAnchorMarker`, `retireBoltAnchorMarker` and `seedOneRepositoryBoard`, and a path deleted and recreated within one process."
  - "`oneRepositoryAt` against the Goal: a missing path answers two-repository without logging, and an answer for a worktree on disk is cached per worktree. Is the `.git` Lstat check right for linked worktrees (a `.git` file) and for errors other than not-exist? Is the cache key sound for symlinked or differently spelled paths?"
  - "`MergeStateOperation` in `recordsduringmerge.go`: in a two-repository hub only a merge counts, as before f06acf39. Does the one-repository branch keep its prior behaviour, since the Goal puts it out of scope? Does the two-repository branch still handle the conflicted-index case and the other callers of `gitOperationAt`?"
  - "The new tests (`TestMergeStateActive` cherry-pick step, `TestOneRepositoryAt_NoWorktreeOnDiskReadsAsTwoRepositoriesQuietly`, `TestCommitStatusSeam_Real` addition): do they cover the two behaviours the Goal names, a status commit during a weft cherry-pick in a two-repository hub that commits and a missing-path layout read that logs nothing? Do they fail against the pre-fix code? Is the cache covered, given the mutation survivors at `onerepository.go:50`, `:54` and `:61` and the uncovered `forgetLayouts` calls?"
  - "Test isolation: the process-wide `layoutAnswers` map can leak answers between tests that reuse a path. Does any existing or new test depend on a clean cache?"
  - "Mutation survivor `anchormarker.go:85`, the `forgetLayouts()` call in `commitAnchorMarker`: can a test reasonably assert the next layout read sees the new marker, or is it a pure cache invalidation no test can kill?"
  - "Doc comments in `onerepository.go`, `mergestateactive.go`, `recordsduringmerge.go` and `doc.go`: does each say what the code now does, and do `doc.go` and any docs the repository's constraints name land with the change? Check the commit messages and `_lym/landing/summary.md` for claims the diff does not bear out."
---

Seed focus for round 1. The answer key is the decision record's `## Goal` alone, with no plan.
