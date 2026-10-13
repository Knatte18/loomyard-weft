---
round: 2
focus:
  - "In internal/battencli/wire.go, check where the InnerRunDeps build-change seam (ChildBuildChanged) obtains the on-disk lyx path: does it call os.Executable per check, or use a path resolved once at wiring time, and does wire return the resolve error?"
  - "In internal/loomcli/drivewiring.go, check how the hand-over successor spawn (SpawnSuccessor, spawnDetachedDrive) obtains the executable path: does it call os.Executable at spawn time, or take the path resolved in goDriverDepsFor, and does a test pin that the spawn is handed the wiring-time path while the start and resume spawn seam still works?"
---

Both entries concern the same hazard: a deploy replaces the running binary, so a path resolved afterwards may no longer exist.
