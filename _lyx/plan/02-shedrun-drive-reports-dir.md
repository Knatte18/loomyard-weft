# Card 2 — shedrun-drive-reports-dir

**Create:**
- `plan:internal/shedrun#DriveReportsDir` -> `func DriveReportsDir(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#DriveReportsRel` -> `func DriveReportsRel(l *lyxcwd.Location, runID string) string`

**Edit:**
- `internal/shedrun/paths.go`
- `internal/shedrun/paths_test.go`
- `cmd/lyx/notransients_test.go`

**Uses:**
- `internal/shedrun#RunDir`
- `internal/shedrun#StatusRel`

**Intent:**
Give drive reports a durable home under the run directory, declared by `shedrun` since it is the sole declarer of the `shed` segment (Shed Run-Directory Invariant).

- `DriveReportsDir(l, runID)` returns `RunDir(l, runID)` joined with `drive-reports`.
- `DriveReportsRel(l, runID)` returns the anchor-relative form, built the same way `StatusRel` is (`lyxdirs.LyxDirName`, the `shed` segment, `runSegment(l, runID)`, `drive-reports`), for a fabric commit pathspec.
- Declare the `drive-reports` segment once as an unexported constant in `paths.go` and use it in both.
- Update `paths.go`'s file header, which lists its constructors by kind; describe the set rather than pinning a count.

Tests in `paths_test.go`: `DriveReportsDir` equals `RunDir` joined with `drive-reports`, and `DriveReportsRel` equals the anchor-relative suffix of `DriveReportsDir` under both an unanchored and a subpath-anchored location.
In `cmd/lyx/notransients_test.go`, add `shedrun.DriveReportsDir` to `durableSet`, so the guard proves it sits under `_lyx`.

**ImpactSummary:** Two new path constructors in shedrun and one new durable-set guard row; no existing caller changes.
