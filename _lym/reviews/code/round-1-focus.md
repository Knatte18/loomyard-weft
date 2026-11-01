---
round: 1
focus:
  - "internal/dobbyengine/runnotice.go noticeCondition: settle whether the grace check sits where it should relative to the earlier returns and the `memory.Since` episode stamp. Does `memory.Since` hold the running episode's start on the first tick that sees the child, or can it be zero or stale, so the awake-time grace is wrong?"
  - "The grace returns an empty finding and an empty activityStamp. Settle whether that leaves the quiet and agent-based conditions, or the episode memory and marker clearing, in a wrong state compared with the earlier dead-driver return."
  - "internal/battencli/dobbynotice.go childDriverLockHeld: settle whether `loomengine.LoomBootstrapLock` is the lock `lym loom start` actually holds, and whether runLockHeld treats a missing lock file as not held. Settle whether the new loomengine import in battencli breaks a package layering rule."
  - "Scope against the board entry: the entry offers the bootstrap lock, a grace, or reordering the lock in `lym loom start`. Check that taking both of the first two is licensed, and that no change outside the notice path rides along."
  - "Tests: check that the new grace rows, and the existing rows moved past the grace, cover the behavior without duplicating each other. Settle whether the bootstrap-lock row duplicates an existing TestChildDriverLockHeld row."
  - "Check the claims in _lym/landing/summary.md against the diff, in particular that a machine sleep and a pause do not count toward the grace, and the `doc.go` wording."
---

Small diff in the dobby notice path: a grace in `runnotice.go` plus a bootstrap lock in `dobbynotice.go`.
