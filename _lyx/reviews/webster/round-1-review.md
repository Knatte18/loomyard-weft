---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/battencli/wire.go:1080
    summary: batten's ChildBuildChanged calls os.Executable on every check, after a deploy has replaced the running binary, so the binary re-step can never fire
    fix: In battencli's wire (internal/battencli/wire.go), call os.Executable once before the InnerRunDeps literal and capture the path in a local variable, as goDriverDepsFor does with its executable; make the ChildBuildChanged closure call buildvcs.OfFile on that captured path. Return the resolve error from wire when os.Executable fails, as goDriverDepsFor does.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/drivewiring.go:358
    summary: the hand-over successor is spawned through spawnDetachedDrive, which calls os.Executable in the old process after the on-disk binary was replaced, so the spawn can start a path that no longer exists
    fix: Change spawnDetachedDrive to take the executable path as a parameter instead of calling os.Executable. In goDriverDepsFor, pass the already-resolved executable to SpawnSuccessor (a closure over the executable local). Keep the c.spawnGoDriver seam for start and resume, which run in fresh processes, by having its production closure call os.Executable and pass the result. Add a row to TestGoDriver_Settle or TestDriveVerb asserting SpawnSuccessor is handed the path resolved at wiring time.
---

### [MEDIUM] Batten's build check re-resolves the executable after the deploy it is waiting for

**Location:** internal/battencli/wire.go:1080
**Issue:** `ChildBuildChanged` runs `os.Executable()` inside the closure, once per due check, and reads the build of the path it returns.
Its own comment says a deploy replaces the lyx on disk under this running batten.
On Linux, `os.Executable()` reads `/proc/self/exe`, and for a running binary whose file was unlinked or renamed over, that link reads `<path> (deleted)`.
`buildvcs.OfFile` then fails on that path, `binaryRestep` logs the failure as a skipped check, and no halted child is ever re-stepped.
This depends on how the deploy replaces the file, which I could not confirm from `update-plugins.sh` here; a deploy that rewrites the file in place would not hit it.
The Go driver's wiring avoids the hazard by resolving `executable` once in `goDriverDepsFor` ("the on-disk lyx path is resolved once, here"), so the two seams disagree on how they find the on-disk build.
The test over `newInnerRunDeps` fakes cannot see this, since it fakes the seam.

### [MEDIUM] The hand-over spawns its successor from a path resolved after the binary changed

**Location:** internal/loomcli/drivewiring.go:358
**Issue:** `handOver` runs only when the on-disk build differs from the one this driver is running.
It then calls `SpawnSuccessor`, which is `spawnDetachedDrive`, which calls `os.Executable()` at that moment.
With the same `(deleted)` behavior as the finding above, the exec of the old path can fail, and `handOver` only logs a Warn and leaves the run parked, so the hand-over silently degrades to a hand-back every time a deploy replaces the file by rename.
`goDriverDepsFor` already holds the path it resolved before the replacement, and uses it for `OnDiskBuild`, so the information the spawn needs is in hand.
As in the finding above, whether the deploy replaces the file this way is unconfirmed; the fix costs nothing either way.

## Rejected

- Repair cap counts every `restep` record of the stop key, including the tackler's and arbiter's resumes.
  This matches card 10 (`repairCount` counts `restep` settle records of the stop key) and a restarted driver keeps the cap, so it is not a defect.
- A halted driverless go child re-stepped twice for one episode and build.
  The marker keys on the stop key and build and is written only after a successful spawn, and after the re-step the child's last step records the new build, so `changed` is false; not reachable.
- The applied mark written ahead of the move.
  `shedengine.Relaunch` runs `CheckGotoTarget` before `Admit`, and `Admit` is the last step before the write, so the mark follows every refusal and lands under the one status-lock hold; matches card 8.
- Per-card mechanical checks: no reference to `recordMatchesHalt` or `battenChildDriver` remains in the tree, and the budget test, the package tests of every touched package and the seam tests pass.
- Batten-only InnerRun seams: all five new ones are wired non-nil in `internal/battencli/wire.go` and listed nil-legal in `internal/testkit/envkit/envkit.go`.
