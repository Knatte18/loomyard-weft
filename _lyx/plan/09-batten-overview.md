# Card 9 — batten-overview

**Create:**
- `plan:internal/battencli#battenCLI.overviewCmd` -> `func (c *battenCLI) overviewCmd() *cobra.Command`
- `plan:internal/battencli#overviewRun` -> `type overviewRun struct`
- `plan:internal/battencli#overviewSource` -> `type overviewSource struct`
- `plan:internal/battencli#newOverviewSource` -> `func newOverviewSource(prime *lyxcwd.Location, now func() time.Time) *overviewSource`
- `plan:internal/battencli#overviewSource.frame` -> `func (s *overviewSource) frame() statuspane.Frame`
- `plan:internal/battencli#renderOverview` -> `func renderOverview(runs []overviewRun, now time.Time) []string`
- `plan:internal/battencli#fitOverview` -> `func fitOverview(lines []string, rows int) []string`
- `internal/battencli/overview_test.go`
- `internal/battencli/overview_integration_test.go`

**Edit:**
- `internal/battencli#Command`
- `internal/battencli#battenCLI.resolvePersistentPreRun`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/statuspane#FormatMinutes`

**Intent:**
Build D2's runs overview as `lyx batten overview`, a hidden pane verb (audience internal, a cobra child of `batten`, a `Short`, JSON errors per `PATTERN-cli-cobra`) in `overview.go`.
`resolvePersistentPreRun` skips it as it skips `cleanup`: it takes no slug and arms no shed spec.
Its body resolves the cwd (`lyxcwd.CwdFrom`, `lyxcwd.Resolve`), refuses outside the prime through `RefuseUnlessPrime` on the JSON envelope, builds the prime's reed engine (`reedengine.LoadConfig`, `hubgeom.ReedGeometry`), and runs a `statuspane.Pane` on stdout with `statuspane.TerminalSize(os.Stdout)`, `statuspane.ReportFromEnv(os.Getenv, engine.SetContentRows)`, `time.Sleep` and the source's `frame`.
Bound: it only reads the prime's run records and the pairs' status files, without their locks, and never blocks or slows a run; its one write is the content-rows report, which card 3's checks confine to the `runs` strand.

`overviewSource` holds the prime, a `statuspane.StatusCache`, the decoded seeds by slug (a seed is write-once, so a decoded one is never read again), and the pair locations it resolved, and it resolves a child's status path through a `childStatus func(slug string) (string, error)` field that defaults to `lyxcwd.ResolveWorktree` on the pair directory and `shedrun.StatusFile(<pair>, shedrun.SelfRunID)`, kept per slug while the pair exists, so a test substitutes a temp path.
Each `frame` call lists the prime's runs (`shedrun.List`), decodes each seed (`shedrun.ReadSeed`) and skips one whose recipe is not `shedrun.RecipeBatten`, polls batten's status (`shedrun.StatusFile(prime, slug)`) through the cache, checks the pair directory `fabricengine.WorktreePath(prime, slug)` with one `os.Stat`, and, while batten's current producer is `battenrecipe.NameRunShed`, polls the child's status through the cache.

The `overviewRun` values it builds, and `renderOverview`'s rules over them (D2):
- A run is shown while batten's state is not done and its pair exists or its current producer is `battenrecipe.NameWorktreeCreate`; a run with a seed and no batten status file is shown as `starting` with no stage; halted and failed runs with a pair show; a done run and a run whose pair is gone do not.
- A status file unreadable on two polls in a row (`StatusPoll.Unreadable`), batten's or the child's, shows the slug and `status unreadable`; one failed poll keeps the line built from the cache's previous status; neither stops the other lines.
  A run whose very first poll fails has no previous status, so its line is the one a seed without a status file gets, `starting`, until the second failure.
- The stage is the child's current producer while batten is in Run-Shed and the child's status is found, and batten's own current producer otherwise, also when the child's file does not exist yet or any more.
- One line per run, columns padded to the frame's widest value: the slug, batten's state, the stage, the time in that stage (from the end of the last history entry of the status the stage came from, falling back to batten's, then to the seed's `started_at`), the run's age (from the seed's `started_at`), and the pair's reed session name, `reedengine.SessionName(fabricengine.WorktreePath(prime, slug))`; an unknown time renders `-`, and times render through `statuspane.FormatMinutes`, so a line changes at most once a minute and the pane's wait can back off.
- Lines are ordered by the seed's `started_at`, then by slug; runs with no readable seed or a seed without `started_at` come last, by slug.
- No run in flight renders the one line `no runs in flight`.
`fitOverview(lines, rows)` keeps the oldest runs, drops the newest first, and ends with `… <N> more runs not shown`, so the pane never exceeds the height reed gives it and the summary is right whatever count reed last applied.

`docs/overview.md`'s batten bullet names the overview verb and that the orch start runs it (card 10).

Tests (`overview_test.go`):
- `TestRenderOverview`, table-driven over built `overviewRun` values and a fixed `now`: two runs ordered by `started_at` with their six fields; a run with no `started_at` and one with no readable seed after them, by slug; a run in Worktree-Create with no pair shown; a seeded run with no status shown as `starting`; a non-done run whose pair is gone hidden and a failed run with its pair shown; a run in Run-Shed whose child status is not found showing batten's producer, never `status unreadable`; an unreadable status showing `status unreadable` beside a normal line; no runs giving `no runs in flight`; and two rows that pass `renderOverview`'s output through `fitOverview` with a told row count: three lines into two rows keep the oldest line and a summary naming one run not shown, and lines that fit come back unchanged.
- `TestOverviewSource_ReadsThePrimesRunsFromDisk` over a temp prime (`locationkit.Location`) with run directories written through `shedrun.WriteSeed` and status files, a pair directory, and a temp child status path behind `childStatus`: a run whose status turns undecodable keeps its line for one frame and shows `status unreadable` on the second; a run in Run-Shed reads its stage from the child file and falls back to batten's producer once that file is removed; a seed with another recipe is not shown.
- `overview_integration_test.go` (`//go:build integration`, it builds a hub through `hubforge`): `TestOverviewCmd_RefusesOutsideThePrime` runs `overview` through `RunCLIIn` from the hub's task pair, wanting exit 1 and an `"ok":false` envelope naming the prime (the `RefuseUnlessPrime` refusal), and from a temp directory that is no git worktree, wanting exit 1 and the resolution refusal on the envelope; neither starts a pane.
- `test-budget.yaml`: `internal/battencli` 69 → 72.

**ImpactSummary:** one new hidden batten verb and its pure rendering; no existing batten verb or producer changes.
