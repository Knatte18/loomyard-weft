---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/hubgeom/doc.go:14
    summary: hubgeom doc still says ReedGeometry is the one teller that does I/O and the only parent reader, but BurlerGeometry and WebsterGeometry now call ResolveParent too
  - id: F2
    severity: NIT
    class: consistency
    location: internal/hubgeom/webstergeom.go:26
    summary: the resolve-parent-or-warn block is copied verbatim into BurlerGeometry and WebsterGeometry instead of living in one helper
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/usage_test.go
    summary: TestCompactedSince_IgnoresBoundaryAtOrBeforeSince duplicates the "at the newest boundary" row of TestCompactedSince_FindsNewestMainChainBoundaryAfterSince
---

Range: `status.json` gives `product.parent` `main`, and no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet ./...` and `go test ./...` pass on the branch.

Focus-directive verdicts:

- Cards 4–9: every `shuttleengine.Spec` built by a spawning module sets `Skills`, and every opening stencil in `roleOpeningStencils` carries the marker.
  The parent name comes from `hubgeom.ResolveParent` through the reed, burler and webster geometry, and an empty name renders the no-parent variant.
  The spec literals without `Skills` (`awaitLiveJudge`, `awaitLiveSeed`, `probeLiveRound`) are attach probes that match on output files only, and `shedrecipe`'s generic `SingleLLM` row is used by no recipe.
  No role is missed.
- Cards 10–11: the only `ly-drive` hits outside `_lyx/`, `.lyx/` and `docs/research/` are the `bootstrap_test.go` fixture strand name, which the card keeps; the driver launches from `shed-template-driver` through `driverprompt.go`.
- Cards 13 and 15: both cycle modes pass through the handoff-requested note gate before `/clear` or `/compact`.
  The reload sequence runs after a clear, after a watcher compaction and after an auto-compaction, reading `CompactedSince` against `CompactionBaseline`.
  Each typed step is gated on an idle probe on the same tick.
  Not a defect.
- Card 2: `loadSkill` in `shuttleengine/run.go` bounds every path (turn end, unknown notice, timeout, died pane), so a reaction that ends no turn is skipped at the timeout.
  Not a defect.
- Card 16: both PATTERN entries are present and backed by `roleskills_test.go` and `parentdirective_test.go`; the `doc.go` files of every changed package that has one were updated except `hubgeom` (F1).
- Test economy: one duplicate (F3); every added behavior found has coverage.

### [LOW] hubgeom doc contradicts the code

**Location:** `internal/hubgeom/doc.go:14`
**Issue:** the doc says "ReedGeometry is the one teller that does I/O" and that it is "still the only reader" of the parent.
`BurlerGeometry` and `WebsterGeometry` now read the origin record through `ResolveParent` as well.
The CLAUDE.md same-commit docs rule requires the module doc to describe the changed behavior.
**Fix:** reword the paragraph so all three tellers that resolve a parent are named, keeping the told-geometry sentence about `reedengine`.

### [NIT] Duplicated parent resolution

**Location:** `internal/hubgeom/webstergeom.go:26`, `internal/hubgeom/hubgeom.go:110`
**Issue:** the same eight-line block resolves the parent, logs a warning on error and falls back to empty, differing only in the engine named in the log message.
**Fix:** one unexported helper in `parent.go` that takes the engine label and returns the name.

### [LOW] Duplicate test

**Location:** `internal/shuttleengine/claudeengine/usage_test.go`
**Issue:** `TestCompactedSince_IgnoresBoundaryAtOrBeforeSince` asserts that a boundary at `since` is not found, which the `at the newest boundary` row of `TestCompactedSince_FindsNewestMainChainBoundaryAfterSince` already covers, at three chunk sizes.
**Fix:** delete the redundant test.
