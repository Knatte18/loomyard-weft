# Round 1 fixer report

All three findings are fixed, each in its own commit.

- F1 (LOW): `internal/hubgeom/doc.go` now names `ReedGeometry`, `BurlerGeometry` and `WebsterGeometry` as the parent readers, and says an unresolvable parent leaves `ParentName` empty with a warning in all three.
- F2 (NIT): `parentNameOrEmpty` in `internal/hubgeom/parent.go` replaces the copied resolve-or-warn block in `reedGeometry`, `BurlerGeometry` and `WebsterGeometry`; the warning text is unchanged.
- F3 (LOW): `TestCompactedSince_IgnoresBoundaryAtOrBeforeSince` is deleted; the `at the newest boundary` row of `TestCompactedSince_FindsNewestMainChainBoundaryAfterSince` covers it.

`go build ./...`, `go vet` and `go test` on `internal/hubgeom` and `internal/shuttleengine/claudeengine` pass after the fixes.

## Deferred

Nothing.
