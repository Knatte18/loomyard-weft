---
round: 1
focus:
  - "internal/lock/tracked.go and internal/lock/lock.go: does the helper mirror a tracked file's path under the checkout's .lym/locks/, and how does it behave for a path outside the checkout, a relative path, or a symlinked or junctioned path?"
  - "Remaining `path + \".lock\"` and `.swaplock` sites across internal/ (fabricengine, shuttleengine, reedengine, adoptcli, shedadapters, sheddriver, boardengine, gitrepo): does any site whose file sits under _lym/ or Bolt still build a lock beside the file?"
  - "The check that refuses a lock path built beside a file under _lym/ or Bolt: does it cover the sites the Goal lists, and can it be bypassed by a differently spelled construction?"
  - "internal/fabricengine and internal/gitrepo exclude handling: with the `*.lock` and `*.swaplock` excludes removed, does any code path still create or leave a lock file in a tracked tree, and does an existing checkout carrying the old excludes or stray locks stay correct?"
  - "Lock directory creation under .lym/locks/: is the parent directory made before the lock is taken, in every moved site, including internal/state/state.go and the Bolt push lock paths?"
  - "pattern/ entries and PATTERN.md: does the diff land the declared PATTERN changes with the declared text?"
---

Seed focus for round 1, drawn from the Goal's four Idea bullets and the diff's file list.
