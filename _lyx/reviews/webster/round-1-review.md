---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/reedengine/doc.go:571
    summary: the new orch-pane bullet is hard-wrapped at a fixed column, breaking PATTERN-comment-line-breaks
  - id: F2
    severity: NIT
    class: consistency
    location: internal/orchengine/state.go:72
    summary: CycleCount comment still says cycles that reached /clear, but a completed compaction increments it too
---

Range: `main` is the run's parent and no rework round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Build, `go vet` (default, `smoke` and `integration` tags over the touched packages) and the unit and integration tests of the touched packages pass.
The card groups are Edit and Create only, so no Rename or Delete mechanical check applies.

Focus items settled with no defect:

- `usage.go`: `readContext` drops the window's first line unless the window starts the file, doubles the window until a qualifying entry or the file start, and scans lines last to first, so the first hit is the later of the latest main-chain assistant entry and the latest main-chain `compact_boundary`. An entry on a chunk edge is read whole once the window widens.
- `tickIdle`: the re-read runs after every gate, skips a requested cycle, applies the hard cap for a hard trigger and the soft threshold for a soft one, and never falls through from hard to soft on the same tick.
- `tickCompacting`: completion needs `BoundaryAt` at or after `PhaseEnteredAt` plus a passing idle probe, timeout returns to idle with `compaction timed out` and sets `LastDeferral`, and an unconfirmed `/compact` is re-typed only when idle with no qualifying boundary.
- Compact versus clear: the D8 hold is applied to a hard trigger only in compact mode and never to a requested one; DEFER handling and handoff writing are not reached in compact mode.
- `LoadConfig` still goes through `LoadOrTemplate`, refuses other permission modes and unknown cycle modes with the fix named, and resolves an empty `permission_mode` to `bypass`.
- Notice queue and batten wiring: only the watcher calls `Send` for a notice, delivery is idle-gated, `battenshed` queues through `Notify` without importing `orchengine`, and the queue path is declared once in `internal/orchcli/paths.go`.
- Stencils, adopt pid reuse and smoke tests match cards 10 to 12.

### [BLOCKING] Hard-wrapped doc comment in reedengine

**Location:** `internal/reedengine/doc.go:571`
**Issue:** the added bullet "The orch pane under the same policy" wraps mid-sentence at about one hundred columns, which PATTERN-comment-line-breaks forbids for new comments; the neighbouring wrapped bullets are legacy and not a convention to match.
**Fix:** rewrite the bullet with one sentence per line and a break at an independent-clause boundary, leaving the untouched neighbouring lines alone.

### [NIT] Stale CycleCount comment

**Location:** `internal/orchengine/state.go:72`
**Issue:** the field comment says "Cycles that reached /clear", but `tickCompacting` increments `CycleCount` on a completed compaction and the status output reports it for both modes.
**Fix:** reword the comment to cover a clear that was typed and a compaction that completed.
