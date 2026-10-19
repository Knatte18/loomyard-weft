# Card 3 — reconcile-launchers-restored

**Edit:**
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#Topology.repairPairWiring` -> `func (t *Topology) repairPairWiring(rec *Mutations, warpLayout *lyxcwd.Location, slug, warpBranch string, pr *ReconcilePairResult, setAction bool)`
- `internal/fabriccli#Command`

**Delete:**
- `internal/fabricengine#restorePortalAndLaunchers`
- `internal/fabricengine#ReconcileActionPortalRestored`
- `internal/fabricengine_test#TestReconcile_RestoresDeletedPortalAndLaunchers`

**Create:**
- `plan:internal/fabricengine#ReconcileActionLaunchersRestored` -> `const ReconcileActionLaunchersRestored ReconcileAction = "launchers_restored"`
- `plan:internal/fabricengine#restoreLaunchers` -> `func restoreLaunchers(rec *Mutations, warpLayout *lyxcwd.Location, slug, warpBranch string, pr *ReconcilePairResult) bool`
- `plan:internal/fabricengine_test#TestReconcile_RestoresDeletedLaunchers` -> `func TestReconcile_RestoresDeletedLaunchers(t *testing.T)`

**Prosa:**
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Uses:**
- `internal/fabricengine#LauncherDir`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#appendPrDetail`
- `internal/fabricengine#ReconcileActionAlreadyHealthy`
- `internal/gitrepo#Repo.BranchMarkedLocal`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`
- `internal/fslink#CreateDirLink`

**Intent:**
Reconcile stops restoring the pair's portal and reports a launcher restore as `launchers_restored` (D6).

`restoreLaunchers` replaces `restorePortalAndLaunchers`.
It skips the prime, as today, and skips a pair whose warp branch carries the local-only mark (`gitrepo.Repo.BranchMarkedLocal` on the warp repository), since `AddLocal` creates no launcher directory.
A mark read that fails appends its raw read error to the pair's Detail, under `refusal-spec`'s transient exemption for a propagated raw I/O error, and restores nothing.
Otherwise it recreates a missing launcher directory through `writeLaunchers`, noting `launcher scripts restored` in the Detail and reporting whether it restored.
A restore failure stays a Detail note, never an Error or a changed Action.
It creates no portal link.
`repairPairWiring` takes the warp branch `Topology.Reconcile` already read and passes it on; when `restoreLaunchers` restored and the Action is still `already_healthy`, the Action becomes `ReconcileActionLaunchersRestored`, so a consumer keying off Action still sees that convergence altered the pair.
`ReconcileActionPortalRestored` goes.

Tests:
- `TestReconcile_RestoresDeletedLaunchers` (integration) replaces `TestReconcile_RestoresDeletedPortalAndLaunchers`.
  - An ordinary pair whose launcher directory was deleted reports `launchers_restored` and has its directory back.
  - A second Reconcile reports `already_healthy`.
  - A pair added with `AddLocal` reports `already_healthy` after Reconcile, and no launcher directory exists for it.
  - A pair with both its launcher directory and a warp junction deleted keeps `junction_repointed`, and its Detail notes the launcher restore.
  - With `_launchers/<AnchorRel>` replaced by a directory link (`fslink.CreateDirLink`) to a directory outside the hub, so `writeLaunchers`' `os.Root` mkdir refuses on every platform, a pair whose launcher directory was deleted reports `already_healthy` with a `launcher restore failed` Detail and no Error.

Docs in this card:
- The `lyx fabric reconcile` help text in `internal/fabriccli#Command` says reconcile restores a missing launcher directory and reports `launchers_restored`, with no portal.
- `SANDBOX-FABRIC-SUITE.md`'s F17 expects `launchers_restored` for a deleted launcher directory and no portal restore.

**ImpactSummary:** One reconcile action value is renamed to `launchers_restored` and the portal restore goes; `repairPairWiring` has one caller, and only the reconcile help text and sandbox suite name the old value.
