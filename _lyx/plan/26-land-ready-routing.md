# Card 26 — land-ready-routing

**Create:**
- `plan:internal/battenshed#ReadyRoute` -> `type ReadyRoute struct`
- `plan:internal/orchcli#NotifyOrchAt` -> `func NotifyOrchAt(worktree *lyxcwd.Location, program, line string) (bool, error)`
- `plan:internal/battencli#readyRoutes` -> `func readyRoutes(ready []string, parentBeam func(slug string) string, beamOrchLive func(beam string) (bool, error)) (prime []string, beams map[string][]string, err error)`
- `plan:internal/battencli#TestReadyRoutes` -> `func TestReadyRoutes(t *testing.T)`

**Edit:**
- `internal/battenshed#LandDeps`
- `internal/battenshed#landProducer.Call`
- `internal/battenshed#rowNotifier.ready`
- `internal/battenshed#TestLand_ReadyNotice`
- `internal/battencli#landDeps`
- `internal/orchcli#TestParentOrchStrand`

**Extends:**
- `internal/battenshed/doc.go`
- `contracts/stencils/orch/orch-template-role.md`

**Uses:**
- `internal/boardengine#ParentBeam`
- `internal/orchcli#OrchLive`
- `internal/orchcli#OrchStrandOf`
- `internal/battencli#notifyRunParent`

**Intent:**
D13's exception: the Land row's ready notice takes no walk.
`LandDeps` gains `ReadyRoutes func(ready []string) ([]ReadyRoute, error)`; a `ReadyRoute` carries a destination key, the slugs it names and its own notify seam.
A nil `ReadyRoutes` keeps today's notice, every newly ready slug riding the landing line, so the existing Land tests, which set none, are unchanged.
`rowNotifier.ready` posts the landing line through the walk as today (`<landed>: landed`), naming after `newly ready:` only the slugs of the route whose key is the prime's when the walked destination is the prime, and posts one `<landed>: landed; newly ready: …` line per other route to that route's destination, each once per batten run under its own marker line, so a resumed row sends none twice.
`battencli.landDeps` fills `ReadyRoutes` through `readyRoutes` over the board as the Land row reads it: a newly ready task whose `ParentBeam` is empty goes to the prime's orch (`NotifyOrchAt` of the prime), and one whose `ParentBeam` names a beam goes to that beam's own orch (`NotifyOrchAt` of the beam's worktree) only when the beam's worktree is present on this machine and its orch is live by `OrchLive`.
Every other ready task, its beam's worktree absent here or its beam's orch stopped or never started, is named in no ready notice, since no orch it would reach may start it: its `parent_beam` keeps it out of every other orch's filter.
Bound: such a task waits unnoticed until the operator or the beam's orch, once started, reads the board; the beam's own orch on another machine gets no notice of it either.
`NotifyOrchAt` posts to the named worktree's orch strand (`OrchStrandOf`) with no walk, and posts nothing, with no error, when that worktree records no strand.
`TestReadyRoutes` rows: a prime task goes to the prime; a member of a live beam goes to the beam; a member of a beam whose orch is stopped, and one whose beam worktree is absent, go nowhere; a liveness error is returned.
`TestParentOrchStrand` (card 20) gains a row: `NotifyOrchAt` of a worktree whose orch state records no strand posts nothing and returns false with no error.
`TestLand_ReadyNotice` rows: a beam route is posted as its own line once, a resumed row posts no route twice, and the prime route rides the landing line.
The orch role stencil says a ready notice names only tasks the receiving orch may start.

**ImpactSummary:** The Land row's ready notice splits by destination; LandDeps gains one seam that battencli's landDeps is the only production filler of.
