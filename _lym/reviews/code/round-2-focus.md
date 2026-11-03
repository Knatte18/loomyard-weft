---
round: 2
focus:
  - "internal/fabricengine/add.go rollbackAdd passes the leftover into removeWeftWorktree: settle whether a test drives a failed Add with a refilled weft directory and asserts the directory is gone and Add's error returned."
  - "internal/fabricengine/destroy.go finishGitWorktreeRemoval: settle whether a test fails if the isRegisteredLinkedWorktreeIn guard is dropped, using a sibling that git refuses to remove while it stays registered, and whether the gitErr-plus-finish-failure path has a sibling row."
  - "internal/fabricengine/destroy.go finishGitWorktreeRemoval: settle whether a failed worktree listing can still license a recursive delete of a worktree git refused to remove, and whether the clean-exit failure message names a way forward."
---

Targets the areas of round 1's changes, so the reviewer judges the fix text fresh.
