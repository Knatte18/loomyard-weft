# Card 20 — plan-write-archives-webster

**Edit:**
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomshed/doc.go`
- `internal/loomshed/planwrite_test.go`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe/entries_planwrite_test.go`
- `PATTERN.md`
- `pattern/PATTERN-plan-generation.md`

**Uses:**
- `internal/loomshed#stalePlanRotation`
- `internal/loomshed#rotateStalePlanDir`
- `internal/websterengine#ArchiveRunRecord`

**Intent:** A loom `goto` back to Plan-Write retires the Webster run record with the plan generation it belongs to, so the next Webster run starts a new run instead of looping on `plan_drifted` (D9).
`NewPlanDirRotator` takes a told `archiveWebster func(dest string) error` (the same seam PR-Rework is handed, which loomcli's wiring backs with `websterengine.ArchiveRunRecord` over the task's webster geometry), and `planWriteEntry` passes `env.Rework.ArchiveWebster`, required through the registry's existing seam check.
When the rotation has stale plan files to move, the closure first calls `archiveWebster` with `<archive-dir>/webster` (the subdirectory name PR-Rework's round layout already declares in loomshed, reused rather than spelled again), then moves the plan files; archiving first means a failure leaves the plan in place, so a retry renders and rotates again, and an already-moved record is a no-op on the retry since `ArchiveRunRecord` skips an absent webster dir.
`ArchiveRunRecord` creates the destination only when a run record exists, and refuses while a run holds the run lock, which aborts the rotation before the plan moves.
The next Webster run starts from the current HEAD, keeping the old run's commits, since the new plan was written against that tree.
Bound: the archive runs only inside the rotation, only when stale plan files are being rotated, and moves files without deleting or resetting anything; the Plan-Review and Webster-Review run directories stay out of it.
`PATTERN.md`'s `PATTERN-plan-generation` line and its background file state that Webster's run record also moves with Plan-Write's `archive-*/` rotation, in the same commit as the code.
`planwrite_test.go` extends `TestNewPlanDirRotator`: a rotation calls the seam once with `<archive-dir>/webster` before moving the plan, a rotation with nothing to move never calls it, and a seam error leaves every plan file in place; `entries_planwrite_test.go` covers the required seam; the loomshed package doc names the archive.

**ImpactSummary:** Adds one told seam to NewPlanDirRotator, threaded from its one registry entry; the rotation's own file moves are unchanged.
