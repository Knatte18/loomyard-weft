---
round: 1
focus:
  - "internal/fabricengine/export_test.go: both sides appended helpers; check that the task's lock-wait seam exports and the parent's new hub-workspace and portals-sweep exports do not collide or leave a stale name, and that lockwait_test.go still compiles against the merged helpers."
  - "internal/fabricengine/doc.go: the task edited the seed-commit and lock paragraphs, the parent rewrote the portals and workspace-file paragraphs; check the merged text is consistent with PATTERN-hub-containment's retired `_portals` and hub-dir workspace file, and names no `--empty=drop` or removed symbol."
  - "internal/fabricengine/destroy.go: the parent widened the geometry-root ownership check to `portalsDirName`, the task reworded `errDropReplayFailed`; check the parent's retired-portals sweep still reaches destroy through the chokepoint and that the drop path's error wording is not asserted by a parent-added test."
  - "docs/overview.md and cmd/lyx: check the merged module table and execution stack describe neither retired portals nor a removed verb, and that cmd/lyx/main.go and spawnobservability_test.go still agree after the parent's command-surface changes."
  - "Weft write-lock acquisition: the task routes merge verbs through `acquireWeftWriteLock`; check that parent-added reconcile, sweep or hub-workspace code taking the same lock neither bypasses the wait signal where a merge-verb path is meant nor takes it in an order that can deadlock with the merge verbs."
  - "PATTERN.md and pattern/ changes: check the task's new code (cherry-pick fallback, lockWaitHook, reed row changes) against `PATTERN-fabric-write-containment`, `PATTERN-hub-containment` and `PATTERN-hub-suffix` as the parent changed them."
  - "Test economy: check whether the parent's new reconcile and sweep integration tests and the task's merge-lock tests cover the shared lock and seed-commit paths twice, and whether any interaction behavior is left uncovered."
---

Seed focus from the overlapping files and the parent's pattern changes; the reviewer judges the interaction, not either side alone.
