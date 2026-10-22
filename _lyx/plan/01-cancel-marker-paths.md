# Card 1 — cancel-marker-paths

**Create:**
- `plan:internal/shedrun#CancelMarker` -> `func CancelMarker(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#CancelMarkerLock` -> `func CancelMarkerLock(l *lyxcwd.Location, runID string) string`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun/doc.go`

**Intent:**
Declare the batten cancel marker's two paths in `shedrun`, the sole declarer of run-directory paths (PATTERN-shed-run-directory, D3).
`CancelMarker` returns `<runSegment>.cancel.json` and `CancelMarkerLock` returns `<runSegment>.cancel.lock`, both joined directly onto `ScratchRoot(l)`, using the same run-segment alias resolution `ScratchDir` uses.
They sit beside the per-slug scratch directories, never inside `RunDir` or `ScratchDir`, so batten cleanup's per-slug removal (D6, which empties `ScratchDir` and removes `RunDir`) never touches them, and `cleanupSlugs`, which lists only directories under the scratch root, never reads them as a slug.
`TestPathConstructors` gains rows pinning both paths for a slug and for the `self` alias, and asserting neither lies under `ScratchDir(l, runID)` or `RunDir(l, runID)`.
`doc.go` names the cancel marker and its lock beside the tackler marker's paragraph: written by `lyx batten cancel`, read by batten's verbs, the lock being the writer's liveness.

**ImpactSummary:** Two new constructors and one extended table test in `shedrun`; no existing caller changes.
