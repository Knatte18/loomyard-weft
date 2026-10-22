---
round: 2
focus:
  - "internal/loomcli/wiring.go `wire`: does the `websterGeom` copied into `runDeps.Geom` carry a non-nil `VerifyHold` built from the same `PauseGeometry` the rest of `wire` reads, so the loom run's webster verify gate tells `Site.Hold` on its first run and rerun as the declared `PATTERN-verified-tree` text requires?"
  - "internal/testkit/envkit/envkit.go nil-legal map and `TestWire_EverySeamFilled`: does the suite fail if `wire` leaves `WebsterDeps.Geom.VerifyHold` nil?"
  - "internal/websterengine/verifygate.go `NewVerifyGate`: does a test in internal/websterengine drive a failing first run then a passing rerun with a non-nil `Geometry.VerifyHold` and assert both verify calls received that hold, and does a nil-hold case run the gate as before?"
---

The merged `wire` is where the task's verify-gate hold meets the parent's `wiring.go` changes.
