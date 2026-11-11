---
round: 2
exclude_lenses: []
focus:
  - "Freshness and selection in D4 and D5: settle whether the map's staleness reference is consistent with where maps are built (the prime, at main's HEAD) and with the tier base D1 names, so that a worktree whose base predates a landing, or whose tier base is a branch commit, does not fall back to whole-package runs in common flows."
  - "D2's per-covered-package hashes and format marker: settle whether a map written by an older binary, or built from a prime working tree with a modified or untracked Go file in a covered package other than the tested one, can be mistaken for a valid new-shape map."
  - "D3 and D1 bounds: settle whether each let-through (tests the spawn scan cannot see, or a changed function that is no key) states what it skips and what bounds it."
---

Focus is on the areas the BLOCKING and design-class MEDIUM findings touched, so the fresh round checks the reworked text there.
