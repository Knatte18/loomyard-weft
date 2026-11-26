# Card 9 — reconcile-before-load

**Create:**
- `plan:internal/hubgeom#ReconcileBeforeLoad` -> `func ReconcileBeforeLoad(l *lymcwd.Location)`
- `plan:internal/loomcli#TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads` -> `func TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.arm`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig`
- `internal/webstercli#websterCLI.wireHub`
- `internal/shuttleparity#loomConfigOf`
- `internal/hubreconcile#reconcileWayForward`
- `internal/hubreconcile#EnsureOrWarn`
- `internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/beamcli/doc.go`
- `internal/webstercli/doc.go`
- `internal/shuttleparity/doc.go`
- `internal/hubreconcile/doc.go`

**Intent:** Every lym verb outside `lym config` that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load (D5), so the first such verb on a new build moves the hub's old keys before a strict load would refuse them.

`hubgeom.ReconcileBeforeLoad(l)` is the one call every such site makes: it runs `EnsureOrWarn` over `ReconcileGeometry(l)` when `l` is in a hub, and does nothing outside one.
- `loomCLI.resolvePersistentPreRun` calls it for every `lym loom` subcommand, before its skip of the circling, decision, parent-review and peers groups and the checklist and pusher verbs, resolving the location from cwd; a cwd that is no git repository runs no walk and the pre-run goes on as today.
  That reaches every loom verb, `arm`'s and the self-resolving groups' alike, so `perchHardCap`'s loads in the circling group are covered.
  `loomCLI.arm` drops its own start-and-resume call, which the pre-run now makes for every verb.
- `hubLandDeps` calls it before its `CodeReviewCaps` closure first loads loom's config, `wireHub` before it loads loom's config, and `loomConfigOf` before its two loads, each from the location it already has.
- The `lym config` verbs run no walk: they are card 2's ways forward, and `lym config reconcile --apply` reconciles its own layer.
Once the stamp names the running build, the call costs the stamp read and the board file checks a start verb pays today.
Bound: the walk runs once per build per machine, at the first verb that loads the loom or tapestry config; while a worktree stays mid-merge or the board pull skips the hub-wide write, the stamp stays stale, so every such verb reruns the walk, taking the hub lock and pulling the board, until the merge ends or the board is brought up to date.
A file the walk has not moved keeps its old key, and that verb's load refuses it under card 2, naming its layer's reconcile command.

`reconcileWayForward` adds re-running the verb once the walk ends beside the start verbs it names today:
`fix the cause the error names, then re-run this verb once the reconcile can finish, or run a start verb to retry it: ...`.
`EnsureOrWarn`'s doc says the next verb that loads a reconciled config retries the walk.

Tests:
- `TestEnsureOrWarn_WarnsWithTheWayForward` (tier1) asserts the new way forward.
- `TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads` (tier1, new, in a tier1 test file of `internal/loomcli`): over a `hubforge` hub with a stale build stamp, whose hub base and a pair's local `loom.yaml` hold `code_max_bounces: 2`, `lym loom circling` run through the loom command tree in the pair reads the code perch's caps with `loom.code.max_bounces` resolved to 2 and does not halt; the same verb from a cwd outside any git repository refuses as today with no walk.
- `TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig` (tier1) row: with a stale stamp and a hub base holding `code_max_bounces: 7`, `CodeReviewCaps` reads 7.

**ImpactSummary:** One new hubgeom helper called by the loom pre-run and three other load sites; `arm` loses its own call, and the failed-reconcile warning's way forward changes text; outside a hub, and with a current stamp, nothing changes.
