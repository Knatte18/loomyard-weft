---
round: 2
focus:
  - "internal/fabriccli/hubmutation_integration_test.go: does any surviving test assert the positive `refusal` object (non-empty check, what, target, reason; check equal to fabricengine.CheckContainment), the flattened `error` and the `mutations` key of a `lym fabric remove` failure envelope driven by a real gate refusal?"
  - "internal/fabricengine/retiredlaunchers_test.go, TestSweepRetiredLaunchers: does a row pin a symlink at `_launchers` itself, so that replacing `root.Lstat` with `Stat` in `SweepRetiredLaunchers` would fail a test, and does the row also pin the `.cmd` names in `retiredLauncherFileNames`?"
  - "internal/fabricengine/doc.go and destroy_test.go: does any comment still cite the deleted `refuseUncontainedPath`, and does `hubRel` now live in a file matching its callers?"
---

Targets the round-1 test-coverage gaps and the dangling references after their fixes.
