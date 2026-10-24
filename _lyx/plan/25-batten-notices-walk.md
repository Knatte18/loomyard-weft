# Card 25 — batten-notices-walk

**Create:**
- `plan:internal/battencli#notifyRunParent` -> `func notifyRunParent(prime *lyxcwd.Location, slug, line string) (bool, error)`
- `plan:internal/battencli#runParentRecorded` -> `func runParentRecorded(prime *lyxcwd.Location, slug string) (bool, error)`
- `plan:internal/battencli#noticeSite` -> `func noticeSite(prime *lyxcwd.Location, slug string, pair func() (*lyxcwd.Location, bool, error), parentOf func(slug string) (string, error)) (site *lyxcwd.Location, isPair bool, err error)`
- `plan:internal/battencli#TestNoticeSite` -> `func TestNoticeSite(t *testing.T)`
- `plan:internal/battencli#TestNoticesReachDriversParent` -> `func TestNoticesReachDriversParent(t *testing.T)`

**Edit:**
- `internal/battencli#rowNotices`
- `internal/battencli#noticesReachDriversParent`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.driverDepsFor`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/battencli#hubCancelDeps`

**Delete:**
- `internal/battencli#orchStrandRecorded`

**Uses:**
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`
- `plan:internal/orchcli#NotifyParent`
- `plan:internal/orchcli#NotifyFrom`
- `plan:internal/orchcli#ParentOrchRecorded`
- `plan:internal/orchcli#OrchLive`
- `plan:internal/hubgeom#ParentOf`
- `plan:internal/hubgeom#NearestLiveOrch`

**Intent:**
D13 for batten: batten's notices are sent from the prime's location (D8) but act on a run's pair, so each delivery takes the walk from that run's pair, never from where batten runs.
`noticeSite` picks the walk's start at each call: the run's pair when it is present here (the walk then starts at its recorded parent, `orchcli.NotifyParent`), else the worktree the pair forks or forked from, `hubgeom.ParentOf(prime, slug)`, when that worktree is present here (`orchcli.NotifyFrom`), else the prime.
`notifyRunParent` posts through that site, and `runParentRecorded` answers whether the walked destination records an orch strand, the fact the watched marker and the quiet rules read in place of `orchStrandRecorded`, which is deleted.
`rowNotices`, the InnerRun and Land notice seams in `wire`, the driver hand-back and tackler notices (`driverDepsFor`, `loadedTacklerDeps`) and the cancel notice (`hubCancelDeps`) take them.
`noticesReachDriversParent` asks whether the walked destination equals the destination the task driver's own parent message reaches, the nearest live orch from the pair's recorded parent, so a driver and batten under a live beam orch both reach it.
`TestNoticesReachDriversParent` over a hubforge hub with a recorded shortname, a marked beam `b` with a member pair `m`, a prime pair `p`, and orch states the test seeds with no reed session behind them, so no beam orch reads live: `noticesReachDriversParent` answers true for `p` and for `m` with the prime's orch state recording a strand (both reach the prime), and false with none recorded; `runParentRecorded` answers the same for `m`'s slug, and for an absent pair of `b`'s member with `b`'s worktree present.
`TestNoticeSite` rows over fakes: a present pair is the site, an absent pair of a present beam starts at the beam, an absent pair whose beam is absent here starts at the prime, a task of the prime starts at the prime, and a resolver error is returned.

**ImpactSummary:** Every batten notice seam takes the walked destination; one prime-only helper is deleted with its callers moved on the same card.
