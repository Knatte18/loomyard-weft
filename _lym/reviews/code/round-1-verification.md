---
round: 1
entries:
  - key: reconcile-commits-unloadable-profiles
    status: closed
    finding: F1
    reason: "e6c2d8241, e9593c80e, 5f9b37fb0: ReinterpretArrasDefault drops a defaults.arras naming a profile with no arras block, is registered on the profiles module and applied by the hubreconcile walk after the follow, and an upgrade-path test asserts Load succeeds"
---
