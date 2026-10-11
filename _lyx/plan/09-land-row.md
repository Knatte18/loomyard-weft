# Card 9 — land-row

**Create:**
- `plan:internal/shedrun#LandLock` -> `func LandLock(l *lyxcwd.Location) string`
- `plan:internal/battencli#LandLock` -> `func LandLock(l *lyxcwd.Location) string`
- `plan:internal/battencli#readySlugs` -> `func readySlugs(tasks []boardengine.Task) []string`
- `plan:internal/battencli#runDeploy` -> `func runDeploy(ctx context.Context, primeDir, command, logPath string, timeout time.Duration) (battenshed.DeployResult, error)`
- `plan:internal/battencli#TestReadySlugs` -> `func TestReadySlugs(t *testing.T)`
- `internal/battencli/deploy_integration_test.go`
- `plan:internal/shedrecipe#landEntry` -> `func landEntry(name string, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/battenrecipe#NameLand` -> `const NameLand = "Land"`
- `plan:internal/battenrecipe#TestRecipeEngines_ReportsEveryEngineNameSortedOnce` -> `func TestRecipeEngines_ReportsEveryEngineNameSortedOnce(t *testing.T)`
- `plan:internal/battenrecipe#TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun` -> `func TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun(t *testing.T)`

**Delete:**
- `internal/battenrecipe#TestRecipeEngines_ReportsExactlyTheFourEngineNamesSorted`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestLocksAreDistinct`
- `internal/battencli#TestPaths_ResolveUnderTheAnchorBySegment`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#preGoto`
- `internal/battencli#TestPreGoto`
- `internal/battencli#haltedOverDoneChild`
- `internal/battencli#TestHaltedOverDoneChild`
- `plan:internal/battencli#TestWire_BattenStatusSeamsReadBattensOwnStatus`
- `internal/battencli#fourRowRunForRecipe`
- `internal/battencli#stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords`
- `internal/battencli#shortPollBattenRecipe`
- `internal/shedrecipe#Env`
- `internal/shedrecipe/entries_batten.go`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#lifecycleEntryCases`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/battenrecipe#battenRowEngines`
- `internal/battenrecipe#TestNew_OnDoneChainAndLoadBearingEdges`
- `internal/battenrecipe#TestNew_RowNamesMatchTheDurableIdentityConstants`
- `internal/battenrecipe#TestNameRunShed_ValueIsPinnedAgainstASymmetryRename`
- `contracts/recipes/batten-recipe.yaml`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/battenshed#NewLand`
- `plan:internal/battenshed#LandDeps`
- `plan:internal/battencli#rowNotices`
- `internal/landingshed#LoadConfig`
- `internal/verifyrun#Run`

**Intent:**
The batten recipe gains the Land row between Run-Shed and Worktree-Teardown (D1), registered, wired and guarded.
The registry entry, the recipe row and the wiring land together, because the cross-consumer coverage guard in `internal/shedrecipe` fails on a registered engine no recipe reaches, and `battenrecipe.New` fails on a recipe row naming an unregistered engine or an Env missing its seams.

Recipe: `contracts/recipes/batten-recipe.yaml` gains `- name: Land`, `engine: Land`, `on_done: Worktree-Teardown`, with no `on_stuck` and no `max_bounces`, and Run-Shed's `on_done` becomes `Land`.
Its header says the row names are five durable identities by name rather than by count, describes the Land row and says a failed Land step halts batten blocked at Land with the pair present, so Worktree-Teardown stays the only terminal and unreachable from any failure path.
A batten status seeded before this change, at Run-Shed, routes into Land on Run-Shed's Done, since routing is read from the embedded recipe at every step.
`battenrecipe.NameLand` joins the row-name constants, whose block comment stops counting the rows, and `battenRowEngines` maps it to `Land`.

Registry: `landEntry` validates `Slug`, `ScratchDir`, `PrimeLock.Acquire`, `PrimeLock.Path`, `Land.LandLock.Acquire`, `Land.LandLock.Path` and the seams `Land.ParentBranch`, `Land.PrimeBranch`, `Land.PrimeDirty`, `Land.Pull`, `Land.Landing`, `Land.RunDeploy`, `Land.ReadyTasks`, `Land.PruneBoard` and `Land.PauseRequested`, rejects any Config key, and returns `battenshed.NewLand(name, env.Slug, env.Land, env.PrimeLock, env.ScratchDir)`.
It lives in `internal/shedrecipe/entries_batten.go` beside the other batten entries, whose shared `Env.Slug`/`Env.ScratchDir`/`Env.PrimeLock` validation shape it has, and that file's header stops counting and naming its entries.
`registry` maps `"Land"` to it, and `Env` gains `Land battenshed.LandDeps`, a whole-struct passthrough documented like `Teardown`.

Paths: `shedrun.LandLock` returns `<AnchorPath>/.lyx/shed/land.lock`, beside `PrimeRunLock` and built the same way from `lyxdirs.DotLyxDirName` (PATTERN-lyxdirs-single-declarer, PATTERN-durable-vs-ephemeral-state), and `battencli.LandLock` forwards to it as `PrimeRunLock` does.

Wiring, in `battenCLI.wire`, every seam resolved on Call:
`ParentBranch` reads the task worktree's fabric origin record (`fabricengine.ReadOrigin`) and returns its parent branch, an absent record being an error; the Land producer decides an empty one (card 8);
`PrimeBranch` and `Pull` open the prime's fabric (`fabricengine.Open`) for `CurrentBranch` and `Pull` with `EnvSyncOptions`;
`PrimeDirty` returns the prime fabric's `DirtyPaths` that lie under the prime's code worktree root, since the records side is pulled non-fatally by fabric's pull and its run records change under every batten;
`Landing` loads `landingshed.LoadConfig` from `fabricengine.BoardDir` and returns its `Deploy` and `RequirePRToBase` (PATTERN-config-strictness);
`RunDeploy` is `runDeploy` over the prime's code worktree root;
`ReadyTasks` opens the hub's board and returns `readySlugs` of `ListTasksFull`, and `PruneBoard` runs the board's `Prune(true)`, the operation of `lyx board prune --apply`;
`LandLock` acquires `LandLock(location)` non-blocking with `lock.TryAcquireWriteLock`, creating its directory first, as the prime lock's closure does;
`PauseRequested` reads batten's own status as Run-Shed's does;
and `Notices` is `rowNotices(location, slug)`.
`readySlugs` returns, sorted, the slugs of tasks of kind `task` with no status (unclaimed) and an empty `depends_on`.
`runDeploy` creates the log file, runs `verifyrun.Run` under a context with the timeout in `primeDir`, and logs the spawn and the wait at Info through `internal/logger` (PATTERN-spawn-observability); a timeout reports `TimedOut`, and a shell that could not start is the returned error.
`preGoto` admits a goto onto Land only on the terms it admits Worktree-Teardown: the batten run already at that row, or a child whose status reads done, so a goto cannot land a child that did not finish.
The first term admits the way forward of a pause honoured in Land (Shared Decision `land-row-pause`), a goto onto Land with batten blocked there.
The existing goto onto Worktree-Teardown with a done child stays admitted and now skips Land's pull, deploy, prune and ready notice: it is kept as the operator's way past a Land step they settled by hand, such as a deploy run by hand after a broken `deploy` command.
What bounds it: it is an explicit operator verb, never a failure route, it still requires a done child or batten already at Worktree-Teardown, and it skips only the landing's prime-side steps, never the child's own Finalize, which already landed the branch.
`haltedOverDoneChild`'s status flag stops saying the run goes straight to Worktree-Teardown: it says `lyx batten run <slug>` from the prime resumes batten at the row it halted at, which lands the run through Land before Worktree-Teardown tears the pair down.

`internal/testkit/envkit`: `FullEnv` fills `Land` with every seam `landEntry` validates, the two lock paths and `Notices.OrchStrandRecorded` and `Notices.BattenHistoryLen`; `nilLegal` lists `Land.Sleep`, `Land.Now`, `Land.LandLock.Sleep`, `Land.Notices.Notify` and `Land.Notices.Sleep`, each a documented nil default.
`docs/overview.md`'s batten entry names the Land row between Run-Shed and Worktree-Teardown: what it pulls, deploys and prunes, the land lock, that a failed step halts it blocked with one notice, and that batten no longer notifies the orch of a done child.

Tests:
`TestPathConstructors`, `TestLocksAreDistinct` and `TestPaths_ResolveUnderTheAnchorBySegment` gain the land lock;
`lifecycleEntryCases` gains a Land case whose zero-seam rows cover every validated seam, run by the four `TestLifecycleEntries_*` tests, and `newTestEnv` fills `Land`;
`TestNew_OnDoneChainAndLoadBearingEdges` asserts Run-Shed routes to Land on Done, Land to Worktree-Teardown on Done, and Land has no `on_stuck`;
`TestNew_RowNamesMatchTheDurableIdentityConstants` includes `NameLand`, and `TestNameRunShed_ValueIsPinnedAgainstASymmetryRename`'s expected row list gains `NameLand` between `NameRunShed` and `NameWorktreeTeardown`;
`TestNames`' `ShipsExpectedEntries` list gains `Land`, and `minimalEntries` gains a `Land` row over `newTestEnv` and `emptyConfig`, so `TestEveryEntry_RejectsUnknownConfigKey` reaches the new entry;
the new `TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun` steps `battenrecipe.New`'s shed with batten seeded at Land with no pause requested, so shed's step-3 pause check passes and calls Land, and with the land lock contended: the fake `Land.LandLock.Acquire` sets `pause_requested` on batten's status during Land's wait, and `Land.PauseRequested` reads that status, so the step halts blocked at Land with the flag still set, `shedengine.Goto` onto Land leaves it paused with the flag cleared, and the next step calls Land again, which returns Done once the lock frees;
`TestRecipeEngines_ReportsExactlyTheFourEngineNamesSorted` is deleted, since its name pins a count, and `TestRecipeEngines_ReportsEveryEngineNameSortedOnce` asserts `RecipeEngines` equals the sorted, de-duplicated values of `battenRowEngines`;
`TestPreGoto` gains rows refusing a goto onto Land with a running child, admitting it with a done one, and admitting it with batten itself at Land;
`TestHaltedOverDoneChild` asserts the reworded flag;
`TestWire_BattenStatusSeamsReadBattensOwnStatus` gains `Land.PauseRequested` rows beside `InnerRun.PauseRequested`'s;
in the integration tier, `shortPollBattenRecipe` gains the Land row between Run-Shed and Worktree-Teardown and its comment stops counting rows, `fourRowRunForRecipe` steps Land after Run-Shed and asserts it returns Done over the hubforge hub, whose prime sits at its remote head with an empty landing `deploy`, with the pair still present until the Worktree-Teardown step, and `stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords` steps Land between Run-Shed's Done and the teardown whose archive it asserts;
`TestReadySlugs` covers a ready task, a note, a claimed task, a done task and a task with a dependency;
`internal/battencli/deploy_integration_test.go`, tagged `integration` since it spawns a shell, asserts `runDeploy` writes the command's output to the log, reports a non-zero exit code, and reports `TimedOut` for a command outliving a short timeout.
`TestWire_EverySeamFilled` then covers the Land seams with no edit.
The new top-level tests raise the rows of `internal/battencli` and `internal/battenrecipe` in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** batten's recipe gains a row every new and in-flight batten run passes through after Run-Shed's Done; the registry, Env, battencli wiring, envkit fixture, goto guard, halted-over-done status flag and every test that pins the recipe's rows or steps through it follow it.
