# Card 23 — plan-write-archives-webster

**Create:**
- `internal/loomshed#ArchivedPlanWebsterDirs`

**Edit:**
- `internal/loomshed#NewPlanDirRotator` -> `func NewPlanDirRotator(anchorPath, stencilsDir string, archiveWebster func(dest string) error, now func() time.Time) func() (string, error)`
- `internal/loomshed/doc.go`
- `internal/loomshed/planwrite_test.go`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe/entries_planwrite_test.go`
- `internal/shedrecipe/fixture_test.go`
- `PATTERN.md`
- `pattern/PATTERN-plan-generation.md`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#ArchiveRunRecord`

**Intent:** A loom `goto` back to Plan-Write retires the Webster run record with the plan generation it belongs to, so the next Webster run starts a new run instead of looping on `plan_drifted` (D9).
`NewPlanDirRotator` takes a told `archiveWebster func(dest string) error` (the same seam PR-Rework is handed, which loomcli's wiring backs with `websterengine.ArchiveRunRecord` over the task's webster geometry), and `planWriteEntry` passes `env.Rework.ArchiveWebster`, required through the registry's existing seam check.
When the rotation has stale plan files to move, the closure first calls `archiveWebster` with `<archive-dir>/webster` (the subdirectory name PR-Rework's round layout already declares in loomshed, reused rather than spelled again), then moves the plan files; archiving first means a failure leaves the plan in place, so a retry renders and rotates again.
A retry picks the next free `archive-<stamp>-N` directory, since the failed attempt created the first; the record already moved stays under the first, and `ArchiveRunRecord` skips the now absent webster dir, so the retry moves the plan alone.
`ArchiveRunRecord` creates the destination only when a run record exists, and refuses while a run holds the run lock, which aborts the rotation before the plan moves.
The rotation thereby reaches one refusal it never had, and `refusal-spec.md`'s loom section gains its row beside PR-Rework's twin: "plan-write archive: webster busy" (a webster run holds its run lock; transient; wait for the run to finish, then re-step).
The rotation surfaces it with that way forward, and `planwrite_test.go`'s seam-error step asserts it.
`ArchiveRunRecord`'s collision refusal gets no row: it fires only when `<dest>/<entry>` already exists, and the rotation always hands it a `<archive-dir>/webster` under a directory `firstFreePlanArchivePath` has just found absent, so no on-disk state reaches it from this caller.
The next Webster run starts from the current HEAD, keeping the old run's commits, since the new plan was written against that tree.
Bound: the archive runs only inside the rotation, only when stale plan files are being rotated, and moves files without deleting or resetting anything; the Plan-Review and Webster-Review run directories stay out of it.

`ArchivedPlanWebsterDirs`, beside `NewPlanDirRotator` in `planwrite.go`, returns the `webster` subdirectory of every `archive-*/` directory under the plan directory that the rotation writes the record into, named through `planparser`'s archive-name declaration and the same subdirectory name, so a reader outside loomshed spells neither literal; card 24's commit pathspec is its reader.

`PATTERN.md`'s `PATTERN-plan-generation` line and its background file state that Webster's run record also moves with Plan-Write's `archive-*/` rotation, in the same commit as the code.
`planwrite_test.go` extends `TestNewPlanDirRotator`: a rotation calls the seam once with `<archive-dir>/webster` before moving the plan, and that directory is the one `ArchivedPlanWebsterDirs` then returns; a rotation with nothing to move never calls it; and a seam error leaves every plan file in place.
`entries_planwrite_test.go` covers the required seam; `newTestEnv` in `fixture_test.go` fills `Rework.ArchiveWebster` with a closure returning nil, so every construction built on it, the `PlanWrite` row of `conformance_test.go`, `entries_gate_test.go`'s `gateCapableEntries` and `entries_planwrite_test.go`'s happy-path and failure tests, keeps constructing; the loomshed package doc names the archive.

**ImpactSummary:** Adds one told seam to NewPlanDirRotator, threaded from its one registry entry, and one exported reader of the archived record's directories; the rotation's own file moves are unchanged.
