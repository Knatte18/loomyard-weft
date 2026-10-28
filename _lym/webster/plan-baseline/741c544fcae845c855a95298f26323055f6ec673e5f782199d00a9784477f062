# Card 1 — orchroute-walk

**Create:**
- `internal/orchroute/doc.go`
- `plan:internal/orchroute#DirName` -> `const DirName = "orch"`
- `plan:internal/orchroute#RulesRel` -> `func RulesRel() string`
- `plan:internal/orchroute#Paths` -> `func Paths(location *lymcwd.Location) orchengine.Paths`
- `plan:internal/orchroute#Live` -> `func Live(worktree *lymcwd.Location) (bool, error)`
- `plan:internal/orchroute#liveFrom` -> `func liveFrom(st orchengine.State, strands func() ([]reedengine.StrandStatus, error)) (bool, error)`
- `plan:internal/orchroute#StrandOf` -> `func StrandOf(worktree *lymcwd.Location) (name string, recorded bool, err error)`
- `plan:internal/orchroute#Prime` -> `func Prime(location *lymcwd.Location) (*lymcwd.Location, error)`
- `plan:internal/orchroute#WorktreeOrPrime` -> `func WorktreeOrPrime(location *lymcwd.Location, worktree string) (*lymcwd.Location, error)`
- `plan:internal/orchroute#From` -> `func From(start *lymcwd.Location, live func(worktree *lymcwd.Location) (bool, error)) (name string, recorded bool, err error)`
- `plan:internal/orchroute#FromPair` -> `func FromPair(pair *lymcwd.Location, live func(worktree *lymcwd.Location) (bool, error)) (name string, recorded bool, err error)`
- `plan:internal/orchroute#Asker` -> `func Asker(location *lymcwd.Location, live func(worktree *lymcwd.Location) (bool, error)) (name string, recorded bool, err error)`
- `plan:internal/orchroute#TestLiveFrom` -> `func TestLiveFrom(t *testing.T)`
- `plan:internal/orchroute#TestAsker` -> `func TestAsker(t *testing.T)`
- `internal/orchroute/testmain_test.go`
- `internal/orchroute/testmain_integration_test.go`

**Edit:**
- `internal/orchcli#orchDirName`
- `internal/orchcli#rulesRel`
- `internal/orchcli#orchPaths`
- `internal/orchcli#OrchLive`
- `internal/orchcli#OrchStrandOf`
- `internal/orchcli#resolvePrime`
- `internal/orchcli#worktreeOrPrime`
- `internal/orchcli#parentOrchStrand`
- `internal/orchcli#nearestOrchStrand`

**Delete:**
- `internal/orchcli#orchLiveFrom`
- `internal/orchcli/live_test.go`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/orchcli#TestPrimePaths_JoinsRuleFilesAndPrimeRoot`
- `internal/orchcli#trackedStrand`
- `internal/hubgeom#NearestLiveOrch`
- `internal/hubgeom#ResolveParent`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#PrimeName`

**Intent:**
Move the orch walk and its liveness and strand lookups below `internal/orchcli` into a new package `internal/orchroute`, so `internal/creelcli` can resolve a `--parent` receiver (card 2) without importing `orchcli`, which imports `creelcli`.
`orchroute` imports only packages already in `go list -deps ./internal/creelcli`: `orchengine`, `reedengine`, `hubgeom`, `fabricengine`, `shuttleengines`, `lymcwd`, `lymdirs` and `agentname`; it imports neither `orchcli` nor `creelcli`.

The bodies move unchanged in behavior:
`Paths` is `orchcli.orchPaths`'s body, with `DirName` and `RulesRel` the orch subdirectory name and rules-file path it joins, so the orch module's paths stay declared once (`orchcli`'s `orchDirName` becomes `= orchroute.DirName`, `rulesRel` returns `orchroute.RulesRel()`, `orchPaths` returns `orchroute.Paths(location)`).
`Live` is `OrchLive`'s body, reading reed through `reedengine.New(..., shuttleengines.EnvScrub())` and the engine's `Status` directly, over `liveFrom`, which is `orchLiveFrom` with the tracked-strand lookup written inline, since `orchcli#trackedStrand` stays for `orchcli`'s own callers.
`StrandOf` is `OrchStrandOf`'s body, `Prime` is `resolvePrime`'s, `WorktreeOrPrime` is `worktreeOrPrime`'s, `From` is `nearestOrchStrand`'s and `FromPair` is `parentOrchStrand`'s.
Each `orchcli` member listed under Edit becomes a one-line delegation to its `orchroute` counterpart, so every `orchcli` caller and `orchcli#TestParentOrchStrand` keep working unchanged.

`Asker` is new and is the one rule D1 and D3 share, batten's run-notice rule (`internal/battencli/notices.go`): when `location` is the prime (its `WorktreeName` equals `fabricengine.PrimeName`) or a beam (its origin record, `fabricengine.ReadOriginFor(location, location.WorktreeName)`, carries `Beam`), it answers `From(location, live)`, the walk starting at that worktree itself; otherwise it is a pair and it answers `FromPair(location, live)`, the walk starting at the pair's recorded parent.
A worktree with no origin record that is not the prime is a pair whose `FromPair` walk ends at the prime, as `hubgeom.ResolveParent` already decides.
An origin read error is returned.

`TestLiveFrom` is `orchcli#TestOrchLiveFrom` moved unchanged into `orchroute` (`//go:build tier0`); `internal/orchcli/live_test.go`, which held only that test, is deleted.
`TestAsker` is new, `//go:build tier1`, over a `hubforge.CopyHub` hub with a beam `b`, a member pair `m` forked from `b` and a pair `p` forked from the prime, orch state seeded through `Paths` and `orchengine.SaveState` as `orchcli#TestParentOrchStrand` does, and a told liveness predicate.
Its rows reach only what `Asker` adds over `FromPair`, whose pair walk `orchcli#TestParentOrchStrand` already reaches through the delegation: `b` itself with its orch live answers its own orch, the walk starting at the beam, where `FromPair` would start at `b`'s parent; `b` with its orch not live answers the prime's; the prime answers the prime's; one pair row, `m` with `b`'s orch live answering `b`'s orch, pins that a pair takes the `FromPair` branch; and with no orch strand recorded at the resolved worktree it answers not recorded.
The package's two TestMain files follow `orchcli`'s pair: `internal/orchroute/testmain_test.go` is `//go:build !tier1` and `internal/orchroute/testmain_integration_test.go` is `//go:build tier1`, each running `gitkit.HermeticGitEnv()` and `tmuxkit.Main`.
`internal/orchroute/doc.go` states the package's purpose, the `Asker` rule and that it sits below `orchcli` so `creelcli` can call it; `docs/overview.md` gains the package in its `internal/` tree.

**ImpactSummary:** orchcli's orch path, liveness and walk helpers become delegations to a new orchroute package; every orchcli caller and test keeps its call, and only TestOrchLiveFrom moves.
