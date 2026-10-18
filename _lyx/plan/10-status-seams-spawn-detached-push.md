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
- `TestCommitRecordsVerb_Success` and `TestCommitRecordsVerb_ErrorEnvelopes` take the new `Commit` shape; the synchronous push dispositions stay pinned.
- `TestCommitStatusSeam_Real` swaps loomcli's `spawnStatusPush` for a recorder and asserts the real seam commits onto the pair and requests one spawn for the pair's location, with no push reaching origin from the seam itself.

`PATTERN.md`'s `PATTERN-push-both-sides` line becomes the declared text: "Pushing run records from Go: the pair's code branch goes along, and a per-transition status push goes through fabric's detached push."
`docs/overview.md`'s `lyx batten cleanup` line, its module-table row for `internal/statuscommit` (which says "push warns") and `internal/cleanupcli/doc.go` say the records push is spawned.
The file header comments of `internal/battencli/commitstatus.go` (the push-warns disposition and its push-rejection rationale) and `internal/battencli/cleanup.go` ("pushes the records side only"), batten's package texts, are rewritten to say the push is spawned.

**ImpactSummary:** Three seams and one verb stop pushing synchronously; `statuscommit.Deps` has exactly these two callers, and `commit-records` keeps its synchronous push.

**Commit:** 10: Status seams spawn fabric's detached push instead of pushing
