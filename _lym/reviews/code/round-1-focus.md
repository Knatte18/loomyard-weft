---
round: 1
focus:
  - "internal/fabricengine/retiredlaunchers.go: does the sweep remove only the hub's `_launchers` directory, stay contained inside the hub, and fail safely on an unresolvable prime, a symlink or junction at `_launchers`, and a missing directory?"
  - "internal/cleanupcli/segments.go and cli.go: does `lym cleanup` list and remove an existing `_launchers` once, refuse outside the prime, and leave other hub content untouched?"
  - "internal/fabricengine add.go, addanchor.go, anchorchange.go, reconcile.go, remove.go, prune.go, destroy.go: is every launcher write, ide-menu write and launcher cleanup gone, with no caller of a deleted symbol left and no rollback or ordering path altered?"
  - "internal/fabricengine/ancestors.go, junctionnames.go, slug.go, portals.go: is the reserved `_launchers` name dropped only where it served launchers, and does slug or junction-name validation keep every other reserved name and rule?"
  - "Deleted tests (launchers_containment, portallauncher, reconcile_stale_registration, destroy_containment_toctou, ancestors_test, hubmutation): does each deleted test cover only removed launcher behavior, or does it also pin behavior that still exists and now has no coverage?"
  - "Test economy for the new retiredlaunchers_test.go and the changed cleanup, add-rollback and livestate tests: duplicates of covering tests, uncovered added behavior, and tests marked `testtiming:keep` that still need it."
  - "docs/, pattern/ and tools/sandbox/SANDBOX-FABRIC-SUITE.md: does any text still describe `_launchers`, ide-menu or launcher scripts as live, and are the docs for the changed fabric and cleanup behavior updated in the same diff?"
  - "Commit messages: any Pattern-Departure trailer, and whether the diff stays within the board entry's scope (no shed, loom or driver code, no fix for the died_driver smoke failure)."
---

Seeded from the board entry and the diff stat: a removal across fabricengine plus a one-time sweep in `lym cleanup`.
