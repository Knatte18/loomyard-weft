# Card 10 — status-seams-spawn-detached-push

**Create:**
- `plan:internal/battencli#spawnStatusPush` -> `var spawnStatusPush func(l *lyxcwd.Location, gateSlug string) (fabricengine.SpawnResult, error)`
- `plan:internal/loomcli#spawnStatusPush` -> `var spawnStatusPush func(l *lyxcwd.Location, gateSlug string) (fabricengine.SpawnResult, error)`
- `plan:internal/battencli#TestStatusPushSpawns_TellTheirLocationAndGateSlug` -> `func TestStatusPushSpawns_TellTheirLocationAndGateSlug(t *testing.T)`

**Edit:**
- `internal/statuscommit#Deps`
- `internal/statuscommit#New`
- `internal/statuscommit#TestNew_OrdinaryPath`
- `internal/statuscommit#TestNew_CommitErrorPropagates`
- `internal/statuscommit#TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`
- `internal/statuscommit#TestNew_PushErrorReturnsNil`
- `internal/statuscommit#TestNew_MergeActiveSkips`
- `internal/battencli#commitStatusDeps`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#newCommitStatusSeam`
- `internal/battencli#TestNewCommitStatusSeam`
- `internal/battencli#cleanupDeps`
- `internal/battencli#fabricCleanupDeps`
- `internal/battencli#cleanup`
- `internal/battencli#CleanupReport`
- `internal/battencli#battenCLI.cleanupCmd`
- `internal/battencli#TestCleanup`
- `internal/battencli#cleanupStubs.deps`
- `internal/battencli#TestCleanupIntegration_RefusesOutsideThePrimeThenDryRunsThenRemovesAndPushes`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#loomCommitStatusDeps`
- `internal/loomcli#newCommitStatusSeam`
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#TestCommitRecordsVerb_Success`
- `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes`
- `internal/loomcli#TestNewCommitStatusSeam_OrdinaryPath`
- `internal/loomcli#TestNewCommitStatusSeam_BoardStatus`
- `internal/loomcli#TestCommitStatusSeam_Real`
- `internal/loomcli#TestLoomPark`
- `internal/loomcli#stubRecordsDeps`
- `PATTERN.md`
- `docs/overview.md`
- `internal/cleanupcli/doc.go`

**Uses:**
- `plan:internal/fabricengine#SpawnStatusPush`
- `plan:internal/fabricengine#SpawnResult`
- `plan:internal/fabricengine#ErrSpawnUnderTest`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#StatusPushLockWait`

**Intent:** Turn every per-transition status push into a spawn of fabric's detached push (D1's first paragraphs), spawning one child per committed transition and none per no-op.

`statuscommit.Deps` becomes `MergeActive`, `Commit func(msg string) (committed bool, err error)` and `Spawn func() error`.
`New` keeps its dispositions in order: skip-while-mid-merge, commit-hard-errors with its re-probe, then, after a nil `Commit`, the after-commit callback as today, and `Spawn` only when `committed` is true.
A `Spawn` error is logged at Warn, "status push spawn failed; the next transition's spawn carries the commits", and the seam returns nil, so a spawn that fails to start warns as a failed push warns today.
Bound: the seam no longer sees a push failure; the child logs it at Warn under `fabricengine.HubLogsDir`, and the next transition's spawn carries the commits.
The package comment and `New`'s comment say spawn where they say push.

Batten: `commitStatusDeps` takes the same `Commit` and `Spawn` shape; `battenCommitStatusDeps`' `Commit` returns `CommitAnchoredPaths`' committed flag, and its `Spawn` calls `spawnStatusPush(location, runID)` from the prime, so the push is the prime's records side only and waits under the pair's pause entry with the run's slug as gate slug; a `SpawnResult.LocalSkipped` is logged at Info.
`newCommitStatusSeam` passes both through, its marker skip unchanged.
`lyx batten cleanup`: `cleanupDeps.Push` becomes `Spawn func() error`, `fabricCleanupDeps` fills it with `spawnStatusPush(location, "")` (the prime, no gate slug, records side only), `cleanup` calls it once when it committed anything, and `CleanupReport.PushError` carries the spawn's failure; the verb's help says the records push is spawned.
Loom and darn: `commitStatusDeps` gains `Spawn`, and its `Commit` returns the committed flag; `loomCommitStatusDeps`' `Spawn` calls `spawnStatusPush(location, "")`, which pushes the pair's records and code sides from a task pair and the records side alone in the prime, the rule `PushPairAnchored` follows today, so a clean-path commit that lands nothing in loom's or darn's seam spawns nothing.
`Push` stays on loom's deps for `commit-records` alone, which keeps its synchronous `PushPairAnchored` with `StatusPushLockWait`; `commitRecordsVerb` reads `Commit`'s error only.
Both packages' `spawnStatusPush` default to `fabricengine.SpawnStatusPush`; a test binary's spawn returns `ErrSpawnUnderTest`, so no test forks a real child, and a test that observes the request swaps the variable.

Tests:
- `TestNew_OrdinaryPath`: a committed transition spawns once after the callback; a nil commit that landed nothing spawns nothing and still runs the callback.
- `TestNew_PushErrorReturnsNil`: a `Spawn` error returns nil and the transition is not retried.
- `TestNew_CommitErrorPropagates`, `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip` and `TestNew_MergeActiveSkips` take the new deps shape and assert `Spawn` is never called on their paths.
- `TestNewCommitStatusSeam` (batten): the marker skip, a committed transition spawning once, and a failing spawn still returning nil.
- `TestCleanup`: a pass that committed spawns once, one that committed nothing spawns none, and a spawn error lands in `PushError`; `cleanupStubs.deps` fills `Spawn` in place of `Push`.
- `TestStatusPushSpawns_TellTheirLocationAndGateSlug`, Tier 1 and serial (it swaps the package's `spawnStatusPush` for a recorder and restores it, while no parallel test runs): `battenCommitStatusDeps(location, "a-slug").Spawn()` requests one spawn for `location` with gate slug `a-slug`, and `fabricCleanupDeps(location).Spawn()` one for `location` with an empty gate slug.
- `TestCleanupIntegration_RefusesOutsideThePrimeThenDryRunsThenRemovesAndPushes`: the removal is committed on the records side locally; its assertions on `push_error` and on the records tip reaching the bare origin go, since the integration binary's spawn forks nothing and the request is pinned by the Tier 1 test above.
- `TestNewCommitStatusSeam_OrdinaryPath` and `TestNewCommitStatusSeam_BoardStatus` (loom): a committed transition spawns once and a no-op none.
- `TestCommitRecordsVerb_Success` and `TestCommitRecordsVerb_ErrorEnvelopes`, through their `stubRecordsDeps` helper, take the new `Commit` shape; the synchronous push dispositions stay pinned.
- `TestLoomPark` takes the new `Commit` shape, its park assertions unchanged.
- `TestCommitStatusSeam_Real` swaps loomcli's `spawnStatusPush` for a recorder and asserts the real seam commits onto the pair and requests one spawn for the pair's location, with no push reaching origin from the seam itself.

The whole plan grows `PATTERN.md` by 371 bytes: this card's `PATTERN-push-both-sides` lengthening (70), card 15's `PATTERN-recipe-setting-lookup` lengthening as card 15 writes it (112) and its new `PATTERN-command-row` line, with no background link, and its newline (189).
`PATTERN.md` is 15556 bytes at the merge base and `internal/pattern`'s `HeadroomBytes` is 15564, so the card gate's headroom lint, measured against the merge base, admits 8 bytes of growth, and a cut of at least 363 bytes is owed before any of that text is added.
This card makes that cut first, 368 bytes, changing only wording outside the three declared entries and keeping every entry's rule, `(test)` marker and background link, so `PATTERN.md` stands at 15188 bytes before any growth and at 15559 after card 15's, 5 bytes under the headroom line.
Line 3's introductory sentence, which is no entry and carries no rule, becomes "Structural invariants, one line per entry: when it applies and what it requires."
Each entry below keeps its name and background link, and its text after the em dash becomes:
- `PATTERN-cwd-resolution`: "Resolving a cwd or worktree root: only `internal/lyxcwd`; nothing else calls `os.Getwd` or `git rev-parse`."
- `PATTERN-told-geometry`: "Writing an engine: handed its absolute paths, it derives none and never imports `internal/lyxcwd`."
- `PATTERN-lyxdirs-single-declarer`: "Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs` declares them, bar one variable block atop a `tools/` script."
- `PATTERN-lyx-temp-root`: "Creating a temp file or dir: under `internal/lyxtemp`'s run dir, never `os.TempDir()`. (test)"
- `PATTERN-durable-vs-ephemeral-state`: "Adding a file: tracked under `_lyx`, never-tracked under `.lyx`, same subpath; nothing a run needs lives only under `.lyx`."
- `PATTERN-hub-containment`: "Linking hub-level containers: from the hub only, never into a worktree."
- `PATTERN-hub-suffix`: "Touching hub naming: `-LYXHUB` is the sole hub container suffix; no code recognises retired `-HUB`. (test)"
- `PATTERN-config-strictness`: "Loading config: a caller uses exactly one of strict `configengine.Load` or `LoadOrTemplate`."
- `PATTERN-binary-change-reconcile`: "Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`, owner of the build stamp and hub lock."
- `PATTERN-refusal-way-forward`: "Adding a webster, shed or loom refusal: its message names the way forward; a test reaches it."
- `PATTERN-fabric-write-containment`: "Writing under `_launchers` or `_portals` from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`."
- `PATTERN-mutation-record`: "Adding a mutating fabric verb: it fills a `*Mutations` record; its result embeds `MutationRecord`."
- `PATTERN-pair-teardown`: "Tearing down a pair: only via `internal/pairteardown`. (test)"
- `PATTERN-batten-bookend`: "Creating or destroying a task worktree: its producer never runs inside it."
- `PATTERN-github-auth`: "Calling GitHub: auth only via `internal/githubclient`; no other production package runs `gh`."
- `PATTERN-agent-filed-issues`: "Filing a GitHub issue: only `lyx selfreport create` calls `selfreportengine.CreateIssue` in production. (test)"
- `PATTERN-gogit-read-helper`: "Reading via go-git in `internal/gitrepo`: only via `readGoGit`, sole caller of `goGit()`. (test)"
- `PATTERN-gitexec-checked-call`: "Running git: `gitexec.Run`/`runChecked`; raw `RunGit`/`r.run` only at pinned `//gitexec:raw` sites."
- `PATTERN-never-force-add`: "Keeping transients unindexed: each repo's own `.git/info/exclude`; fabric and gitrepo never run `git add -f`."
- `PATTERN-parent-lock`: "Merging a parent into a task branch, or landing on it: only under that parent's `internal/parentlock` lock; a moved parent reroutes to Merge-In."
- `PATTERN-shed-producer-seam`: "Importing into `internal/shedengine`: only stdlib, `state` and `lock`; its status and lock paths are caller-supplied."
- `PATTERN-shed-verb-set`: "Adding a run, step, status, pause or goto verb: generic bodies only in `internal/shedverbs`. (test)"
- `PATTERN-treadle-runner-seam`: "Importing into `internal/treadleengine`: only its allowlist, never `burlerengine`, `lyxcwd` or an `internal/*cli` package."
- `PATTERN-verified-tree`: "Running a verify command: only via `verifytree.Verify`. (test)"
- `PATTERN-creel-store`: "Writing the creel journal or reaching a pane: only `internal/creelengine`, a pane only via its `Delivery` seam. (test)"
- `PATTERN-producer-pointer-rule`: "Writing an instruction file: it points at another producer's format contract, never restating it."
- `PATTERN-shuttle-stop`: "Stopping a shuttle strand from Go outside `internal/shuttleengine`: only via shuttle's stop verb. (test)"
- `PATTERN-orch-pane-single-writer`: "Typing into the orch session from Go: only creel, via its `Delivery` seam."
- `PATTERN-shell-guard-table`: "Guarding a shell command: only in `internal/shellguard`, every engine rendering every row; engine tool guards stay in their engine."
- `PATTERN-shell-mechanics-seam`: "Building a pane-shell command string: only via `internal/shell`, which imports only stdlib."
- `PATTERN-role-skills-typed`: "Loading a spawned session's skill: its spawner names it on the launch spec, lyx delivers it on the launch line or typed; no stencil asks for one. (test)"
- `PATTERN-wait-mark-display-only`: "Marking a wait on screen: only reed's `SetWaitMark`; no Go decision reads the marks."
- `PATTERN-leaf-packages`: "Importing into a leaf package: only its closed import set, packages and sets named in the background."
- `PATTERN-sandbox-coverage`: "Registering a lyx module: the sandbox suite exercises it or excludes it with a reason."
- `PATTERN-quarry-cgo`: "Building this module: `CGO_ENABLED=1` and a C compiler, for tree-sitter grammars."
- `PATTERN-testkit`: "Sharing test support across packages: one kit under `internal/testkit/<kit>/`, imported only from tests, never duplicated. (test)"
- `PATTERN-documentation-lifecycle`: "Design docs: none for unbuilt work; a built one lives in its package's `doc.go`."

The card checks the cut with `wc -c PATTERN.md`, which counts bytes, so each em dash counts three, and `go test ./internal/pattern` stays green on it.
With the cut made, `PATTERN.md`'s `PATTERN-push-both-sides` line becomes the declared text: "Pushing run records from Go: the pair's code branch goes along, and a per-transition status push goes through fabric's detached push."
`docs/overview.md`'s `lyx batten cleanup` line, its module-table row for `internal/statuscommit` (which says "push warns") and `internal/cleanupcli/doc.go` say the records push is spawned.
The file header comments of `internal/battencli/commitstatus.go` (the push-warns disposition and its push-rejection rationale) and `internal/battencli/cleanup.go` ("pushes the records side only"), batten's package texts, are rewritten to say the push is spawned.

**ImpactSummary:** Three seams and one verb stop pushing synchronously; `statuscommit.Deps` has exactly these two callers, and `commit-records` keeps its synchronous push.

**Commit:** 10: Status seams spawn fabric's detached push instead of pushing
