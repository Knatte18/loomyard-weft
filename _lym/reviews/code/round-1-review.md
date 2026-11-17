---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/sheddriver/drive.go:352
    summary: The handOver doc and the change description both say a failed hand-over still reaches batten, but batten's binary re-step refuses the dirty tree such a hand-over now leaves behind.
    fix: In internal/sheddriver/drive.go, rewrite the handOver doc line "A failed spawn leaves the run parked with its report, for the parent or a watching batten to pick up." as "A failed spawn leaves the run parked with its report for the parent; a watching batten re-steps it only over a tree with no uncommitted paths outside the run's records." In _lym/landing/summary.md, Design decisions, third bullet, replace "so this failure no longer reaches that path" with "so this failure reaches that path only when the successor spawn fails, where the parked run waits for the parent".
---

### [LOW] A failed successor spawn over a dirty tree is no longer a batten pick-up

**Location:** `internal/sheddriver/drive.go:352`; `_lym/landing/summary.md` (Design decisions, third bullet)

**Issue:** The diff takes the dirty-tree guard out of `handOver`, so a hand-over can now park the run while the tree holds a producer's uncommitted output.
When `SpawnSuccessor` fails there, the run is left parked and driverless at a changed build.
Batten's binary re-step for exactly that state (`internal/battenshed/innerrun_binaryrestep.go:121-130`) still calls `ChildDirtyPaths` and skips the re-step while uncommitted paths exist.
So in that case only the parent can pick the run up.
The doc line "for the parent or a watching batten to pick up" was accurate while the hand-over ran only over a clean tree, and it now overstates batten's part.
The change description's claim that the failure "no longer reaches that path" holds only when the spawn succeeds.
The code is correct; only the two statements are wrong.

## Rejected

- Scope against the Idea (focus 1): the board entry offers three routes, and the third is "a render that names an include its binary lacks hands over to a fresh driver instead of failing the run".
  Removing the guard lets the existing hand-over row (`driverules.go:264`) fire on the failed render's binary-eligible stop.
  That builds the third route, so this is no finding.
  The other path by which the old binary renders new board copies is a render that parses cleanly with the old includes, which the summary names as an open risk.
  That path is not one the entry's third route asks to close.
- Hand-over loop (focus 2): `handedOver` still keys on the stop key and the on-disk build label (`drive.go:336-343`).
  The successor's step records the new build, so `BuildChanged` turns false.
  No loop is possible, and the successor re-steps over the uncommitted output just as the `lym loom resume` route did in the incident.
- Dangling references (focus 3): the build and vet of `./internal/loomcli` and `./internal/sheddriver` pass.
  The imports of `drivewiring.go` are all still used, and `loomengine.PathsOutsideRunRecords` keeps its caller at `internal/battencli/wire.go:1000`.
- Coverage (focus 5): the hand-over row of `TestGoDriver_Settle` pins the spawn, record, order and park.
  The tree is no longer an input to the driver, so the removed row leaves no behaviour uncovered.
- Assistant report: it reports no findings, and the assistant could not run tests.
  Its conclusions agree with the rejections above, and its verdict counts only as advice.
