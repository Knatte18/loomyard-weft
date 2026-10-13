---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/pairteardown/teardown.go:360
    summary: probeQuiet ignores the parent's Go-driver lock, so a settling detached driver counts as quiet and the task's bounded leftover-removal retry meets a long-lived writer it assumes cannot exist
    fix: In probeQuiet, also probe shedrun.DriverLock(task, shedrun.SelfRunID) with the same non-blocking try-acquire runLockFree uses, and report quiet false while it is held, so busyError names the live Go driver and fabric remove refuses with ErrDriverBusy instead of removing a worktree under a live driver. Add one row to the existing probeQuiet test for a held driver lock with a free run lock and loop lock. Optionally reword the decision-record premise comment in internal/fabricengine/remove.go so the late-writer retry claims only short-lived writers after the driver is quiet.
---

### [MEDIUM] Teardown's quiet probe does not see the Go driver's lock

**Location:** `internal/pairteardown/teardown.go:360` (`probeQuiet`), against `internal/shedrun/paths.go` (`DriverLock`, parent) and `internal/fabricengine/remove.go` (`removeWarpWorktreeDir`, task).

**Issue:** The parent added a detached Go driver, `lyx loom drive`, that holds `shedrun.DriverLock` under the worktree's ephemeral scratch directory for its whole life and leaves the run lock free between steps.
`pairteardown`'s `probeQuiet` still defines quiet as run lock free, loop lock free and no live driver strand, and nothing in `internal/pairteardown` reads `DriverLock`.
A Go driver settling between steps therefore reads as quiet, and a detached process is outside the reed session `EndSession` ends.
The task's leftover-directory retry (D7) rests on the premise that, once the session has ended, the remaining writers are short-lived hook or attach processes, so a brief wait clears them.
A live Go driver is a long-lived writer inside the worktree (lock file, settle records, drive reports), so on Windows the sharing-violation retry runs out and the removal fails, and on Linux the worktree is removed under a driver that keeps writing.
Neither side assumed the other: the parent's driver assumes its worktree outlives it, and the task's retry assumes no long-lived writer survives the session end.
Batten waits on `ChildDriverLockHeld` before landing, so the hub path is covered; the operator path (`fabric remove`) is not.

## Rejected

- `drive` run-id handling colliding with `fmt`'s exclusion in `resolveRunID`: `drive` takes `cobra.NoArgs`, so it always addresses the self run, and `fmt` is excluded by name; `drive` correctly keeps the full `wire()`, since it is not in `verbUsesLightweightWiring`.
- Stale wiring-test or `doc.go` enumerations: `TestVerbUsesLightweightWiring` lists `fmt`, the lightweight doc sentence in `cli.go` names it, and `doc.go` documents both `fmt` and the Go driver.
- Refusal rows, stencils and edit directive: the three `fmt` rows are present in `contracts/specs/refusal-spec.md`, the directive line is in `contracts/stencils/edit/edit-directive.md`, and `editdirective_test.go` asserts it.
- Test budget rows: the merged top-level test counts equal the rows for `internal/loomcli`, `internal/battencli`, `contracts/stencils`, `internal/fabricengine`, `internal/shuttleengine` and its `claudeengine`.
- `PATTERN.md` edits: the `PATTERN-role-skills-typed` rewording and the parent's `PATTERN-driver-choice-single-site` change are both present; the task's new code reads no recorded seed driver.
