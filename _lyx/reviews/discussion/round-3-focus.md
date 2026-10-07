---
round: 3
focus:
  - "In decision-record.md D3, settle how the Commit push route (`Fabric.Commit` in `internal/fabricengine/commit.go`, which resolves a `*lyxcwd.Location` from its warp path via `lyxcwd.ResolveWorktree` before classifying paths, and spawns the push with `spawnDetachedPushFn`) learns it runs in the prime, and whether D3's premise about what `Fabric` holds matches that code."
  - "In decision-record.md D3, settle what `Fabric.Commit` does when prime-name resolution fails, given its spawn is fire-and-forget after the commit lands: whether the Decision bounds that failure path so the guard cannot fail open and publish the parent branch, and whether an acceptance criterion covers the Commit route."
  - "In decision-record.md D2, settle whether the origin-branch deletion condition is proven when `Add` in `internal/fabricengine/add.go` skips its pre-flight probe under `SkipGit` or `SkipPush` while step 11's push still runs, so that an unprobed pre-existing origin branch cannot be deleted by rollback."
  - "In decision-record.md D2 and its Constraints, settle whether every statement about recording a refused or failed remote deletion agrees with `PATTERN-mutation-record` (an executor appends only after it observably changed state)."
---

Focus targets the places where the Decisions still leave an unbounded guard or a premise the code contradicts.
