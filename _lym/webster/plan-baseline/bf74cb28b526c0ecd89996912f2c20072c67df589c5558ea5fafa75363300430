# Card 9 — reconcile-before-load

**Create:**
- `internal/hubgeom#ReconcileBeforeLoad`
- `internal/loomcli#TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads`
- `internal/shedcli#TestShedVerbs_StaleStampReconcilesBeforeALoomRecipeArms`

**Edit:**
- `internal/loomcli#loomCLI.arm`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/shedcli#shedCLI.resolvePersistentPreRun`
- `internal/battencli#battenCLI.arm`
- `internal/fabriccli#addMergeVerbs`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#runDobby`
- `internal/beamcli#TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig`
- `internal/webstercli#websterCLI.wireHub`
- `internal/shuttleparity#loomConfigOf`
- `internal/hubreconcile#reconcileWayForward`
- `internal/hubreconcile#EnsureOrWarn`
- `internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward`
- `internal/hubreconcile/doc.go`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/shedcli/doc.go`
- `internal/fabriccli/doc.go`
- `internal/beamcli/doc.go`
- `internal/webstercli/doc.go`
- `internal/shuttleparity/doc.go`

**Intent:** Every lym verb outside `lym config` that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load (D5), so the first such verb on a new build moves the hub's old keys before a strict load would refuse them.

The sites are every route to a strict loom or tapestry load:
`loomengine.LoadConfig` or `LoadTapestryConfig`, `loomcli.ArmAt`, and `configreg.SettingLookup` or `configengine.Load` on the loom or tapestry spec, not the direct `LoadConfig` callers alone.
Each site calls the walk at the verb's entry that first reaches one of those routes.

`hubgeom.ReconcileBeforeLoad(l)` is the one call every such site makes: it runs `EnsureOrWarn` over `ReconcileGeometry(l)` when `l` is in a hub, and does nothing outside one.
- `loomCLI.resolvePersistentPreRun` calls it after its skip of the bare `loom` group and the `checklist` and `pusher` verbs, and before its skip of the circling, decision, parent-review and peers groups, so those groups still walk;
  the one skip condition splits in two around the call.
  It resolves the location from cwd; a cwd that is no git repository runs no walk and the pre-run goes on as today.
  That reaches every loom verb that loads a config, `arm`'s and the self-resolving groups' alike, so `perchHardCap`'s loads in the circling group are covered.
  `loomCLI.arm` drops its own start-and-resume call, which the pre-run now makes for every such verb.
- `shedCLI.resolvePersistentPreRun` calls it after `lymcwd.Resolve` and before `armFromSeed`, for every shed verb it arms; the bare `shed` group and `seed` stay skipped.
  Every recipe's arm, `loomcli.ArmAt` and `battencli.ArmAt` alike, runs after it.
- `battenCLI.arm` calls it for every verb, in place of its `verb == "run"` call, so the batten verbs that reach `configreg.SettingLookup` of `tapestry` walk first.
- The `merge-in` verb's RunE, in `addMergeVerbs`, calls it from `loc()` before `MergeIn`, so the walk runs before the merge changes the worktree and before `websterInFlightWarnings` reads `tapestry.implementer` through `SettingLookup`;
  a walk after the merge would find a conflicted worktree mid-merge and skip it.
- `hubLandDeps` calls it before its `CodeReviewCaps` closure first loads loom's config, `wireHub` before it loads loom's config, and `loomConfigOf` before its two loads, each from the location it already has.
- `runDobby`, the `lym beam dobby` verb's entry, hands `runDobbyWith` a wire seam that calls it with `prime` and then builds the deps through `hubDobbyDeps`.
  A re-exec onto a changed binary re-enters that verb, so every dobby start and every re-exec walks before the dobby's first run notice reads `tapestry.implementer` through `configreg.SettingLookup`;
  `lym beam start`'s walk does not cover a re-exec, which runs no start verb.
  The tests that call `hubDobbyDeps` or `runDobbyWith` directly pass through no walk, so they see no change.
- The `lym config` verbs run no walk: they are card 2's ways forward, and `lym config reconcile --apply` reconciles its own layer.
Once the stamp names the running build, the call costs the stamp read and the board file checks a start verb pays today.
Bound: the walk runs once per build per machine, at the first verb that loads the loom or tapestry config; while a worktree stays mid-merge or the board pull skips the hub-wide write, the stamp stays stale, so every such verb reruns the walk, taking the hub lock and pulling the board, until the merge ends or the board is brought up to date.
The `pusher` and `checklist` verbs and the bare `loom` group run no walk, since they load neither config;
the pusher, which every run commit starts from the hub's Bolt directory, therefore never takes the hub lock.
A file the walk has not moved keeps its old key, and that verb's load refuses it under card 2, naming its layer's reconcile command.

`reconcileWayForward` adds re-running the verb once the walk ends beside the start verbs it names today:
`fix the cause the error names, then re-run this verb once the reconcile can finish, or run a start verb to retry it: ...`.
`EnsureOrWarn`'s doc says the next verb that loads a reconciled config retries the walk.

`internal/hubreconcile/doc.go`'s opening says every verb that loads a reconciled config calls Ensure through EnsureOrWarn, in place of "a start verb calls it".
Its "Skips" section says the next such verb retries a skipped walk, in place of "the next start verb retries", and names the run_settings carry of card 8 among the skip causes.
Its "The walk" section says no verb waits on the board's push, in place of "no start verb waits on a push",
and its sentence that the walk removes no key the running template lacks gains card 6's exception: a key a module's move table names is moved, or dropped when null or when its new path is set.

Tests:
- `TestEnsureOrWarn_WarnsWithTheWayForward` (tier1) asserts the new way forward.
- `TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads` (tier1, new, in a tier1 test file of `internal/loomcli`): over a `hubforge` hub with a stale build stamp, whose hub base and a pair's local `loom.yaml` hold `code_max_bounces: 2`, `lym loom circling` run through the loom command tree in the pair reads the code perch's caps with `loom.code.max_bounces` resolved to 2 and does not halt; the same verb from a cwd outside any git repository refuses as today with no walk.
- `TestShedVerbs_StaleStampReconcilesBeforeALoomRecipeArms` (tier1, new, in a tier1 test file of `internal/shedcli`): over a `hubforge` hub with a stale build stamp, whose hub base `loom.yaml` holds `code_max_bounces: 2`, the test holds the run lock of a pair seeded for a loom-contract recipe, as the `loom step` arm in `internal/shedcli/parity_test.go` does, and runs `lym shed step` there through the shed command tree.
  The verb arms through `loomcli.ArmAt` and its full wire, which loads loom's config strictly, then refuses at `loomPreStep`'s early run-lock probe with kind busy, before the seed commit and `reed.Up()`, so the test starts no tmux server.
  It asserts that the refusal is the busy one and not the moved-key one, and that the hub base `loom.yaml` holds `code.max_bounces: 2` afterwards.
- `TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig` (tier1) row: with a stale stamp and a hub base holding `code_max_bounces: 7`, `CodeReviewCaps` reads 7.

**ImpactSummary:** One new hubgeom helper called by the loom and shed pre-runs, batten's arm, fabric's merge-in, the dobby verb and three other load sites; `arm` loses its own call, batten walks for every verb, the failed-reconcile warning's way forward changes text, and hubreconcile's doc names every loading verb as the caller; outside a hub, and with a current stamp, nothing changes.
