# Card 14 — The orch watcher types the orch's color

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/session.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`

**Uses:**
- `internal/shuttleengine/session.go`
- `internal/segmentcolor/segmentcolor.go`

**Intent:**
Color the orch session like every other strand while keeping the orch watcher the single writer into the orch pane (D14, `PATTERN-orch-pane-single-writer`): the watcher, idle-gated, types the color command as the first step of its start and of every reload, so a `/clear` reload restores it.

Spec (`orchcli/start.go`): the orch's launch spec names `segmentcolor.Coordinator` and sets `ColorByCaller`, so reed tints its tab and border and shuttle types no color for it.

Seam: `orchengine.Session` gains `TypeColor(guid string) error`.
The production adapter (`orchcli/session.go`) reads the orch strand's palette color from its `reedengine.StrandStatus` through the strand seam it already holds, never from reed's config, and plays it with `Runner.TypeColor` (card 11); a strand with no color types nothing and returns nil.

Watcher:

- Start: binding to a strand (`initCursor`) marks the color as pending, in memory; the first tick on which the idle probe passes types it before anything else that tick would type, then clears the mark.
  A restarted watcher types it again, which is harmless.
- Reload: `state.go` gains `ReloadStepColor`, a step ahead of `ReloadStepPlugins`, and `startReload` enters the resuming phase at it.
  Like the plugins step it ends no turn: `typeReloadStep` types it, persists the move to the plugins step and types nothing else on that tick; `reloadStep`'s normalisation knows it.
  A state persisted mid-reload by an older watcher carries no color step and resumes where it was.
- A failed color typing is logged via `logger.Warn` and the sequence moves on; it never wedges a start or a reload, since the color is display only.

Nothing is typed unless the idle probe passed on the same tick, the rule every other watcher write keeps.

Tests:
`watcher_test.go` (fake session) gains rows: a fresh binding types the color on the first idle tick and not before, and only once; a reload types the color first, then the plugins reload on the next idle tick; a failing `TypeColor` still advances the reload; a restart mid-color-step re-types it once idle.
`start_test.go` asserts the orch spec's `Segment` and `ColorByCaller` beside its existing role and focus checks, adding no test function.
`watcher_test.go` holds one top-level function per watcher behavior, so the new behaviors above are new functions: `test-budget.yaml`'s `internal/orchengine` row is set to the package's top-level test count after this card.

`orchengine` `doc.go`: the reload sequence lists the color step first, and the start paragraph names the color typing.

**ImpactSummary:** the orch watcher gains one idle-gated typing at start and one reload step; the orch spec gains two fields.
