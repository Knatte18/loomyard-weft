# Card 2 — verbs-regenerate

**Edit:**
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#Topology.Prune`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#CloneHub`
- `internal/fabricengine_test#slugPermittedRoots`
- `internal/fabricengine_test#checkoutCase`
- `internal/ideengine#TestSpawnScenario`

**Delete:**
- `internal/ideengine#checkPrimeWorkspaceSurvivesTopologyVerbs`

**Create:**
- `plan:internal/fabricengine#regenerateAfterVerb` -> `func regenerateAfterVerb(rec *Mutations, l *lyxcwd.Location, verb string)`
- `plan:internal/fabricengine_test#TestVerbsRegenerateHubWorkspace` -> `func TestVerbsRegenerateHubWorkspace(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine#HubWorkspacePath`
- `internal/fabricengine#Topology.RemoveRefusal`
- `internal/fabricengine#Topology.lockPairTeardown`
- `internal/fabricengine#SetAddFailAfterWarpHookForTest`
- `internal/fabricengine#KindFileWritten`
- `internal/fabricengine_test#TestRemove_FailedWeftTeardownIsReported`
- `internal/fabricengine_test#TestCheckout_JunctionFailureRollsBackBothSides`
- `internal/hubforge#CopyHub`
- `internal/hubforge#NewHub`
- `internal/hubforge#AddPair`
- `internal/hubforge#AddPairWith`
- `internal/testkit/logcapture#Capture`

**Intent:**
Make the fabric verbs that create, remove or switch a warp worktree regenerate the hub workspace file, so it tracks the worktrees on disk (D4).

`regenerateAfterVerb(rec, l, verb)` calls `RegenerateHubWorkspace(rec, l)`.
On an error it logs one Warn, `fabricengine: hub workspace regeneration failed; run "lyx fabric reconcile" once the cause is fixed`, with `verb` and the error, and returns nothing.
A regeneration failure therefore never fails, rolls back or enters the result of the verb; the bound is that a stale or truncated file stays until the next regenerating verb or prime spawn rewrites it.

Call sites, each on every exit past the verb's pre-flight checks, through a `defer` registered at that point after the verb's own `res.Mutations` snapshot defer, so the regeneration's `KindFileWritten` lands in the verb's record:
- `Topology.add` (both `Add` and `AddLocal`): at the line where the pre-flight refusals have passed, just before `disableStoreAutoGC`, so a rollback still regenerates and a pre-flight refusal regenerates nothing.
- `Topology.Remove`: right after `RemoveRefusal` and `inspectPair` succeed, before `lockPairTeardown`, so the regeneration runs after the teardown lock is released; a refused Remove regenerates nothing, and a Remove that fails after taking the task worktree still drops the pair.
- `Topology.Prune`: only when `apply` is true, right after the worktree list is read.
- `Topology.Checkout`: after the pre-flight checks (merge block, weft dirtiness, original-branch capture), before the warp switch, so the rollback path regenerates too and a switch in the prime renames the first folder.
- `Topology.Reconcile`: after the pair loop and the warp-binding backfill, as the repair verb a failed regeneration's Warn names.
- `CloneHub`: on its success path, once the prime layout `l` is resolved, so a fresh hub holds its file; a failed clone tears the hub down and writes nothing.
`Cleanup` is untouched.

Tests:
- `TestVerbsRegenerateHubWorkspace` (integration, external package) reads the file's folder names after each step, and has these rows:
  1. A hub built by `hubforge.NewHub`, whose clone runs `CloneHub`, holds the file at `HubWorkspacePath(h.Location)`.
  2. `Add` of `a-task` adds it, `AddLocal` of `local-task` adds it, `Remove` of `a-task` drops it, and the mutation record of each carries `KindFileWritten` at the file's path.
  3. `Prune(apply)` after deleting a pair's warp worktree by hand drops it; `Prune` without apply leaves the file's bytes unchanged.
  4. `Checkout`:
     - A switch to another branch in the prime renames the first folder to that branch.
     - A Checkout that fails at junction wiring and rolls back, arranged as `TestCheckout_JunctionFailureRollsBackBothSides` does, with the workspace file removed first, recreates the file with the original branch first.
     - A Checkout refused by a dirty weft worktree, with the file removed first, leaves it absent.
  5. With the workspace file removed first, an `Add` failed by `SetAddFailAfterWarpHookForTest` recreates the file, listing the prime and `Bolt` and not the pair.
     This row is serial, because the hook is process-global.
  6. A `Remove` whose weft worktree is locked, as `TestRemove_FailedWeftTeardownIsReported` arranges, returns its error and leaves the file without the pair whose task worktree it took.
     This row is serial and sets `FABRIC_SKIP_PUSH=1` with `t.Setenv`, as that test does, because a failed Remove respawns the pair's push.
  7. Pre-flight refusals regenerate nothing: with the workspace file removed first, an `Add` from a dirty prime and a no-force `Remove` of a pair with a dirty worktree each leave it absent.
  8. With a directory planted at the prime's `.vscode/settings.json`, the regeneration fails before its write.
     An `Add` then succeeds with its usual result, its record carries no `KindFileWritten` for the workspace file, and the captured log holds the Warn naming `lyx fabric reconcile`.
     This row is not parallel because it captures the logger.
     `Reconcile` after the planted directory is removed writes the pair into the file.
- `checkPrimeWorkspaceSurvivesTopologyVerbs` and its step in `TestSpawnScenario` go: they asserted the file survives Remove, which now rewrites it, and rows 2 and 3 cover the verbs.
- `slugPermittedRoots` adds the workspace file's hub-relative path, `HubWorkspacePath` of a location with `RepoName` `warp-bare` and an empty `HubPath`, since a Remove or Prune cell now rewrites it.
- `checkoutCase`'s proceeding expectations (the default state and the warp-dirty either-outcome state) take the same permitted root, since a prime branch switch renames the first folder.

**ImpactSummary:** Six fabric verbs gain a deferred, non-failing workspace regeneration; their results and refusals are unchanged, and the cross-product test permits the one rewritten file.
