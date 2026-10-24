# Card 22 — overview-cleanup-every-orch

**Edit:**
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#TestOverviewSource_DrawsTheOrchLine`
- `internal/cleanupcli#primeSegments`
- `internal/cleanupcli#TestPrimeSegments`

**Extends:**
- `internal/cleanupcli/doc.go`

**Uses:**
- `plan:internal/orchcli#HubOrchs`
- `internal/orchcli#PrimePaths`

**Intent:**
D8's remaining readers of every orch.
`lyx batten overview` draws one orch line per orch of the hub, read on each frame through a new `overviewSource` field `orchs func() ([]orchengine.Paths, []string, error)` (the paths and the slug labels, empty for the prime), which `newOverviewSource` fills over `orchcli.HubOrchs`; a nil `orchs` reads the prime's paths alone, so a hand-built source draws today's one line and the test supplies a fake list, the prime's first and each beam's after it labelled with the beam's slug, each from its own `orchStateCache`, so an unreadable beam state marks only its own line.
`lyx cleanup`'s orch segment runs `orchengine.CleanupHandoffs` over every orch's paths and reports the kept and removed handoffs per worktree.
Test rows: `TestOverviewSource_DrawsTheOrchLine` draws a prime and a beam orch line, and an unreadable beam state broken on its own line only; `TestPrimeSegments` cleans a beam orch's stale handoffs beside the prime's.

**ImpactSummary:** Two readers widen from the prime's orch to a list of orchs; their envelopes gain per-worktree entries.
