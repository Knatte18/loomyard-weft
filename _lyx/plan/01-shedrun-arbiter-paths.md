# Card 1 — shedrun-arbiter-paths

**Create:**
- `plan:internal/shedrun#ArbiterDir` -> `func ArbiterDir(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#ArbiterRel` -> `func ArbiterRel(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#ArbiterAttemptDir` -> `func ArbiterAttemptDir(l *lyxcwd.Location, runID string, attempt int) string`
- `plan:internal/shedrun#ArbiterMarker` -> `func ArbiterMarker(l *lyxcwd.Location, runID string) string`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestDriveReportsRel`
- `internal/shedrun/doc.go`

**Intent:**
Declare the arbiter's paths in `internal/shedrun`, the sole declarer of run-directory paths (D9, `PATTERN-shed-run-directory`), in `paths.go` beside `DriveReportsDir`.

- `ArbiterDir` is the durable directory holding every arbiter attempt of a run: `RunDir(l, runID)` joined with a new unexported segment constant `arbiter`.
- `ArbiterRel` is its worktree-anchor-relative form, built the way `DriveReportsRel` is, so a commit pathspec never names a segment shedrun owns.
- `ArbiterAttemptDir` is one attempt's directory, `ArbiterDir` joined with `attempt-<N>`, where `N` is the attempt's 1-based number.
- `ArbiterMarker` is the ephemeral in-flight marker, `arbiter-inflight.json` under `ScratchDir(l, runID)`, the mirrored `.lyx` subpath (`PATTERN-durable-vs-ephemeral-state`).

No new file spells `_lyx` or `.lyx`: every constructor joins onto `RunDir` or `ScratchDir` (`PATTERN-lyxdirs-single-declarer`).
Add rows for the four constructors to `TestPathConstructors`, asserting the durable ones sit under `RunDir` and the marker under `ScratchDir`, and an `ArbiterRel` row to `TestDriveReportsRel` asserting it is the anchor-relative form of `ArbiterDir`.
Name the arbiter paths in `doc.go`'s list of what the package declares.

**ImpactSummary:** additive path constructors in one package; no existing caller changes.
