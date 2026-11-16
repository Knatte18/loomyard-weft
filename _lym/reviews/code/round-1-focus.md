---
round: 1
focus:
  - "internal/gatecli/testmap.go Long text: the diff drops the sentence about the closing audit round re-measuring the oldest passing entries, yet the audit code (`auditTests`, `audit`, `auditMismatches`) is still live. Does the answer key license that removal, and is the help now false about what the verb does?"
  - "internal/gatecli/testmap.go `coverageMapsDir` and `testMapLocation`: does a non-prime worktree's maps directory under `location.AnchorPath()` resolve correctly for a task worktree, a nested cwd and a prime without a board-level directory? Is `.lym` there ignored by git, so the 'modified or untracked Go file' skip and the HEAD-moved check never see the maps?"
  - "The decision record leaves open whether a call from the prime still writes the hub's shared maps. Does the diff settle that by keeping the prime on the shared maps, and is that consistent across `testmap`, `redundant`, the doc and the overview?"
  - "internal/gatecli/redundancy.go line 282 still reads the hub's shared maps through `testcover.Dir(fabricengine.BoltDir(...))`. Does the answer key's 'lym gate redundant reads the maps from the same place' extend to the redundancy report step, or is the doc's claim that it reads the shared maps from every worktree accurate and intended?"
  - "Build lock and exit record per worktree: does the per-package lock and any exit record in `internal/testcover` or `testmap.go` follow `mapsDir`, so two worktrees never share a lock and a task's lock goes with its worktree?"
  - "Removal of `hubgeom.PrimeWorktreePath`: confirm no caller or doc is left, including docs and tests outside the diff."
  - "internal/gatecli/testmap_integration_test.go: do the new and changed tests cover the non-prime path, the refusal outside a hub, and `lym gate redundant` reading the same worktree's maps? Does either test duplicate the other, and does anything the diff adds go uncovered, such as the `redundant` change?"
  - "Doc and comment claims in the diff (`docs/overview.md`, gatecli `doc.go`, `testcover/doc.go`, `lymdirs/dirs.go`) against the code: is any sentence false, or any consumer of the maps left describing the old prime-only behaviour?"
---

The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
The change description is absent.
