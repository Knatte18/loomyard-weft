# Card 7 — status-push-spawn-entry

**Create:**
- `plan:internal/fabricengine#PushSpawn` -> `type PushSpawn struct`
- `plan:internal/fabricengine#SpawnResult` -> `type SpawnResult struct`
- `plan:internal/fabricengine#SpawnStatusPush` -> `func SpawnStatusPush(l *lyxcwd.Location, gateSlug string) (SpawnResult, error)`
- `plan:internal/fabricengine#statusPushRequest` -> `func statusPushRequest(l *lyxcwd.Location, gateSlug string) (req PushSpawn, localSkipped []string, err error)`
- `plan:internal/fabricengine#PauseSlugFlag` -> `const PauseSlugFlag = "pause-slug"`
- `plan:internal/fabricengine#TeardownSlugFlag` -> `const TeardownSlugFlag = "teardown-slug"`
- `plan:internal/fabricengine#ErrSpawnUnderTest` -> `var ErrSpawnUnderTest error`
- `plan:internal/fabricengine#pushSpawnCommand` -> `func pushSpawnCommand(req PushSpawn) (args []string, dir string)`
- `plan:internal/fabricengine#TestPushSpawnCommand` -> `func TestPushSpawnCommand(t *testing.T)`
- `plan:cmd/lyx#pauseSlugOverride` -> `func pauseSlugOverride(cmd *cobra.Command) *string`
- `plan:cmd/lyx#TestPauseSlugOverride` -> `func TestPauseSlugOverride(t *testing.T)`
- `plan:internal/fabricengine_test#TestSpawnStatusPush` -> `func TestSpawnStatusPush(t *testing.T)`

**Delete:**
- `internal/fabricengine#TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair`

**Edit:**
- `internal/fabricengine#SpawnDetachedPush` -> `func SpawnDetachedPush(req PushSpawn) error`
- `internal/fabricengine#spawnDetachedPushFn`
- `internal/fabricengine#SpawnPairPush`
- `internal/fabricengine#Fabric.Commit`
- `internal/fabricengine#CommitResult`
- `internal/fabricengine#pushCall`
- `internal/fabricengine#pushRecorder`
- `internal/fabricengine#pushRecorder.record` -> `func (r *pushRecorder) record(req PushSpawn)`
- `internal/fabricengine#swapPushRecorder`
- `internal/fabricengine#TestSpawnDetachedPush_SkipEnvAndEmptyPaths`
- `internal/fabricengine#TestCommitLock_ReleasedBeforePush`
- `internal/fabriccli#spawnPush` -> `func spawnPush(l *lyxcwd.Location) (fabricengine.SpawnResult, error)`
- `internal/fabriccli#addWeftVerbs`
- `internal/tendercli#InstallNetworkGate` -> `func InstallNetworkGate(ctx context.Context, pauseSlug *string)`
- `internal/tendercli#TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub`
- `cmd/lyx#newRoot`
- `plan:internal/fabriccli_test#TestRunCLI_LocalPairScenario`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/gitrepo#Repo.BranchMarkedLocal`
- `plan:internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#IsPrimeWorktree`
- `internal/fabricengine#RecordsWorktree`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#SwapPushRecorderForTest`
- `internal/lyxcwd#ResolveWorktree`

**Intent:** Give fabric D1's one `l`-in spawn entry point for a status push, which applies the prime rule, carries the two slug roles, starts the child in `_board` and checks D3's local mark first.

`PushSpawn` is one detached push request: `WarpPath`, `WeftPath`, `PauseSlug`, `TeardownSlug` and `WorkDir`.
`SpawnDetachedPush(req)` keeps its skip-env and both-paths-empty early returns, then returns `ErrSpawnUnderTest` without forking when `testing.Testing()` reports a test binary, since forking it would re-run the test suite; so no test forks a real push child, and a test that needs the request swaps `spawnDetachedPushFn`, whose type becomes `func(PushSpawn) error`.
Its arguments and working directory come from `pushSpawnCommand(req)`, a pure builder: `fabric`, then `--warp-path`, `--weft-path` and `--teardown-slug`, each only when non-empty and each path made absolute, and `--pause-slug` always, written `--pause-slug=` when `PauseSlug` is empty so the prime's empty slug reaches the child, then `push`, and `WorkDir` as the directory.
It starts the child with `cmd.Dir` set to that directory; its spawn log line names the pid, so a stuck child can be ended by it.
`spawn.go`'s file header comment is rewritten to describe the request and the two slug flags.

`SpawnStatusPush(l, gateSlug)` builds the request through `statusPushRequest`, from `l` alone, so no caller names a records path (PATTERN-fabric-vocabulary) or pushes the prime's code branch:
- in the hub's prime (`IsPrimeWorktree`), the records side only, `PauseSlug` the caller's `gateSlug` (empty when it has none) and no `TeardownSlug`, since `Remove` deletes nothing a prime push touches;
- in a task pair, both sides, with `PauseSlug` and `TeardownSlug` both the pair's own slug, `l.WorktreeName`;
- `WorkDir` is always `BoardDir(l.HubPath)`, the hub's `_board` worktree, outside both of the pair's worktrees.
  The child never resolves its hub through the gated `lyxcwd.Resolve`, which refuses `_board` on a hub with a subpath anchor: `lyxcwd.ResolveWorktree` over its working directory, the `_board` worktree root, gives the hub on every hub shape.
Before building anything it reads the mark of each side's current branch with `BranchMarkedLocal`; a marked side is dropped from the request and named in `SpawnResult.LocalSkipped`, and with no side left it spawns nothing.
`SpawnResult.Spawned` reports whether a child was requested.
A failure to resolve the prime or read a mark is returned before any spawn.
`SpawnPairPush` keeps its signature and calls `SpawnStatusPush(l, "")`, returning its error.
`Fabric.Commit` spawns through `statusPushRequest(l, "")` in place of its direct `spawnDetachedPushFn(codePath, weftPath)` call, still only when something landed, and records the skip in a new `CommitResult.PushLocalSkipped []string` (`json:"push_local_skipped,omitempty"`), so a commit in a local pair spawns no child and reports the local branch skipped.
`fabriccli`'s `spawnPush` takes the location and calls `SpawnStatusPush(l, "")`; `lyx fabric sync` appends one `push_spawned` per side the request carried and adds `local_skipped` to its envelope when set.

`addWeftVerbs` registers the two hidden persistent flags `PauseSlugFlag` and `TeardownSlugFlag` beside `--warp-path`/`--weft-path`; bypass mode reads them, and the teardown slug's use lands with card 9.
`InstallNetworkGate` takes `pauseSlug`: nil keeps today's location from `lyxcwd.Resolve` and geometry from `hubgeom.PauseGeometry` over the working directory; a non-nil value resolves the location through `lyxcwd.ResolveWorktree` over the working directory instead, and replaces the geometry's slug with it, empty included, since `_board` no longer ties the child to its pair.
`newRoot`'s pre-run passes `pauseSlugOverride(cmd)` to it: a pointer to the `PauseSlugFlag` value, empty included, when the executing command has that flag and it was set, else nil.
The child's log sink is told its directory: card 9's bypass calls `logger.SetDurableSinkDir(fabricengine.HubLogsDir(hub))` before its first record, so the child logs under `HubLogsDir` on a subpath-anchored hub too, where the cwd-anchored sink does not arm.
What this lets through: a child's pause entry is now whatever its flag says; what bounds it: only `SpawnDetachedPush` sets the flag, from `SpawnStatusPush` or a holder's respawn (card 8), and a hand-run child with a wrong slug waits under that slug's entry, the bound D1 states.

Tests:
- `TestSpawnStatusPush`, over one hubforge hub with `SwapPushRecorderForTest`, rows: from the prime with gate slug `s`, one request with an empty `WarpPath`, the prime's records worktree, `PauseSlug` `s`, no `TeardownSlug`, `WorkDir` the board dir; from a task pair, both worktrees with both slugs the pair's, and `SpawnPairPush` on the same pair records the same request and, with the recorder set to fail, returns that error; from a local pair (`AddLocal`), no request, `Spawned` false and `LocalSkipped` naming both branches; from an ordinary pair whose code branch alone is marked by hand, one request carrying only the records side and `LocalSkipped` naming the code branch; and `Fabric.Commit` in the local pair lands its commit, records no request and reports `PushLocalSkipped`.
- `TestPushSpawnCommand`, Tier 1 table: every field set gives the four flags in order with absolute paths, then `push`, and `WorkDir`; an empty `WarpPath` or `TeardownSlug` omits that flag alone; an empty `PauseSlug` writes `--pause-slug=`.
- `TestPauseSlugOverride`, Tier 1 over a cobra command declaring the flag: set to `child` returns a pointer to `child`; set to empty returns a pointer to the empty string; declared but unset returns nil; a command without the flag returns nil.
- `TestRunCLI_LocalPairScenario` (card 4) gains a step: `lyx fabric sync` in the local pair, with a local commit to sync, exits 0 with `local_skipped` naming both branches and no `push_spawned` entry.
- `TestSpawnDetachedPush_SkipEnvAndEmptyPaths` calls the re-signed form, and gains a row with both paths set and no skip env that returns `ErrSpawnUnderTest`.
- `TestCommitLock_ReleasedBeforePush` swaps the re-typed seam; `pushCall` gains the slug and work-dir fields `record` fills, and `pushRecorder` gains an exported `Err` field its swapped seam returns, nil unless a test sets it.
- `TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair` goes: it builds a `Location` with no repository, which `SpawnStatusPush`'s prime check now refuses, and the request it pinned is `TestSpawnStatusPush`'s task-pair row.
- `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` gains rows, each installed from a hub's `_board` worktree: with pause slug `child`, a network op lands in the hub's pause record marked with slug `child`; with an empty pause slug, it lands under slug `""`, not `_board`; and over a hubforge hub of `Shape{Anchor: "backend"}` with pause slug `child`, the gate installs and the op lands under `child`.

`internal/fabricengine/doc.go` states the entry point, the prime rule, the two slug roles and the `_board` working directory.

**ImpactSummary:** `SpawnDetachedPush` has callers only inside fabricengine and fabriccli, all listed; `InstallNetworkGate` has one production caller, `newRoot`; `SpawnPairPush`'s callers in fabriccli and battencli keep their signature.

**Commit:** 7: fabricengine: one status-push spawn entry with slug roles and the local-mark check
