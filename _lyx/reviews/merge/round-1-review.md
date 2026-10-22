---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/wiring.go:602
    summary: The loom run's in-process webster verify gate is told no pause hold, so a pause never holds its first run or rerun
    fix: >-
      In `wire` in internal/loomcli/wiring.go, set `websterGeom.VerifyHold = verifytree.NewPauseHold(pauseGeom, nil)` after `pauseGeom` is read at line 592 and before `websterGeom` is copied into `runDeps.Geom` at line 639, which means moving the `websterGeom := hubgeom.WebsterGeometry(location)` block below the `PauseGeometry` read or assigning the field after both exist.
      Then delete the `WebsterDeps.Geom.VerifyHold` entry from the nil-legal map at internal/testkit/envkit/envkit.go:83, so `TestWire_EverySeamFilled` fails when `wire` leaves it nil.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/verifygate.go:217
    summary: No test pins that the webster gate tells geom.VerifyHold on both the first run and the rerun
    fix: Add a row to the existing `NewVerifyGate` gate tests in internal/websterengine that wires a `verify` seam recording `site.Hold`, sets `Geometry.VerifyHold` to a non-nil `*verifytree.PauseHold`, drives a failing first run followed by a passing rerun, and asserts both calls received that exact hold; add a nil-hold row asserting the gate runs as before.
    origin: lens:code-tests
---

### [BLOCKING] Loom's webster verify gate runs unheld under a pause

**Location:** internal/loomcli/wiring.go:602 and :639, internal/websterengine/runlevel.go:749, internal/testkit/envkit/envkit.go:83
**Issue:** `wire` builds `websterGeom` from `hubgeom.WebsterGeometry`, which leaves `Geometry.VerifyHold` nil, and passes it as `runDeps.Geom`. Nothing in `loomcli` sets the field. `websterengine.Run` builds the gate with `NewVerifyGate(deps.Geom, ...)`, so in a loom run `site.Hold` is nil on the first run and the rerun. Only the `lyx webster` CLI wiring (`webstercli/wiring.go:229`, `:364`) sets the hold. The declared `PATTERN-verified-tree` text says every site tells `Site.Hold`, and decision-record D5 and its acceptance criterion name the webster verify gate. The `envkit` nil-legal entry hides the gap from `TestWire_EverySeamFilled`, and the `cmd/lyx/verifiedtree_test.go` tripwire matches `.Hold =` in `verifygate.go`, so neither catches it. The gap sits in the task's own commits, so this is not a merge-introduced defect, but the merged `wire` is where it shows and the focus asks whether the gate reaches the told hold. The parent's changes do not alter this path.

### [MEDIUM] Gate first run and rerun hold is untested

**Location:** internal/websterengine/verifygate.go:217, internal/websterengine/doc.go:545
**Issue:** The task changed no websterengine test, and no test there mentions the hold. Deleting `site.Hold = geom.VerifyHold` leaves the suite green. This is the gate first run, rerun and nil-hold interaction the focus names, with no covering test.

## Rejected

- Fork code-tests item "`lyx webster verify` has no test showing it holds under a pause" (LOW): the verb passes `c.geom.VerifyHold` at `verify.go`, `wiring_test.go` pins the field non-nil, and the `verifiedtree_test.go` tripwire pins the site's `Hold:`; a behavioral pause test is the verifytree hold's own coverage, not this interaction.
- Fork code-tests item "envkit nil-legal entry means no test asserts loom's `runDeps.Geom` carries a hold" (MEDIUM): the same defect as F1, whose fix removes the entry; kept inside F1.
- Fork code-correctness items on `refuseCoveredBatch`, `wire` and the parent's `postBatchChecks`: all ran correct, no defect.
- Parent's `doc.go` pack-section removal against the task's verify-gate paragraph: merged cleanly, no surviving sentence describes deleted pack behavior.
- Parent's `wireLightweight` removal of `Env.StencilsDir`: `wire` fills `StencilsDir` at line 915, so the seam test and the merged `Env` are unchanged.
- Parent's changes to `PATTERN-glyph-conversion-chokepoint` and `PATTERN-review-round`, and the new `PATTERN-template-overlay`: the task's code touches none of them.
