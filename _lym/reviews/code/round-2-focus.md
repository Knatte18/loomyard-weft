---
round: 2
focus:
  - "internal/fabricengine/destroy.go `RemoveStrayTaskPath` and `removeWorktreeLeftover`, and `pattern/PATTERN-fabric-git.md` (Destruction chokepoint): does every executor that deletes a path either pass the destructive gate's containment, ownership, dirtiness and force checks, or sit under an exception the PATTERN text names and the code meets, with a `Pattern-Departure:` trailer on any commit that departs?"
  - "internal/battencli/wire.go `teardownRequest`, internal/fabricengine/pairparts.go `PairParts.StrayPath` and `Any()` through `inspectPair`, and internal/battencli/beamland.go `beamMembers` reading a pair through `PairWorktreeStands`: does a test fail if `ClearStray` is dropped, if the `inspectPair` mapping breaks, or if a plain directory reads as a pair?"
  - "internal/fabricengine/remove.go `Remove` for a kept stray directory (`ErrStrayPathKept`), and internal/battencli/cancel.go `removePairAndBranch`: does a kept directory skip the pair push respawn, does a remote step error win over the stray error, and does the `cancel --remove` report show a deleted stray directory?"
---

Round 1 left one design question on the destruction chokepoint and several test-coverage gaps on the stray-directory delete.
