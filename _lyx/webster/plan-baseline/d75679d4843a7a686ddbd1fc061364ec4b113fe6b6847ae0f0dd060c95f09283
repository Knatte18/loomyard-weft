# Card 9 — reconcile-failure-warns

**Create:**
- `plan:internal/hubreconcile#EnsureOrWarn` -> `func EnsureOrWarn(geom Geometry, opts Options)`
- `plan:internal/hubreconcile#reconcileWayForward` -> `const reconcileWayForward = ""`
- `plan:internal/preflightshed#TestPreflight_ReconcileFailureWarnsAndGoesOnToTheCheck` -> `func TestPreflight_ReconcileFailureWarnsAndGoesOnToTheCheck(t *testing.T)`
- `plan:internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward` -> `func TestEnsureOrWarn_WarnsWithTheWayForward(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.arm`
- `internal/battencli#battenCLI.arm`
- `internal/orchcli#Command`
- `internal/preflightshed#preflightProducer.reconcileHub` -> `func (p *preflightProducer) reconcileHub()`
- `internal/preflightshed#preflightProducer.Call`
- `internal/preflightshed/doc.go`
- `internal/hubreconcile/doc.go`
- `internal/hubgeom/doc.go`
- `internal/loomcli#TestLoomResumeReconcilesTheHubConfigBeforeArming`
- `internal/orchcli#TestOrchIntegration_Refusals`

**Delete:**
- `internal/preflightshed#reconcileRefusal`
- `internal/preflightshed#TestReconcileRefusal_NamesTheCauseAndLoomResume`
- `internal/preflightshed#TestPreflight_ReconcileFailureStopsTheRowWithItsWayForward`

**Uses:**
- `internal/hubreconcile#Ensure`
- `internal/testkit/logcapture#Capture`
- `internal/hubreconcile_test#stampKey`

**Intent:**
The operator's addition after the Discussion: a hub whose stamp differs from the binary's templates is reconciled once at its next read, and a failed reconcile leaves the config as it was, writes no stamp, warns with the error and the command to reconcile by hand, and lyx goes on with the existing config, trying again at the next read.
The stamp is `hubreconcile`'s build stamp, whose key folds `configreg.Fingerprint`, which covers every template's text, so a template change makes it stale; a matching stamp costs one comparison, the stamp read and stats `ensure` already makes.
The reads are the start verbs and the Preflight row, the points that call `Ensure` before any module config loads, which keeps `Ensure` the one route (`PATTERN-binary-change-reconcile`); the reconcile that drops the old template's seeded entries is card 8's walk.
What this card adds is the failure half.

`EnsureOrWarn` calls `Ensure` and, on any error, a `*WorktreeError`, a `*LockTimeoutError` or another walk failure, logs one `logger.Warn` naming the error and `reconcileWayForward`, and returns; it never fails its caller.
`reconcileWayForward` is the command to reconcile by hand: fix the cause the error names, then run a start verb, such as `lyx loom resume` in the task worktree, `lyx batten run` or `lyx orch start` from the prime, whose walk retries the reconcile; the error text itself already names the failing file and `lyx config <module>`.
The walk restores the config files of the failing step, the board's with its marker or one worktree's, to their prior bytes and writes no stamp, so the caller goes on with config no failed write touched and the next start verb retries.
Steps that committed before the failure keep their commits, as D3 and `internal/hubreconcile/doc.go` already decide: a failure in a later worktree leaves the committed board, its marker and earlier worktrees' commits in place, which is a reconcile completed for those parts, not a half-written file; the overview's Shared Decisions record this reading of "left as it was".
`loomCLI.arm` for `start` and `resume`, `battenCLI.arm` for `run` and `orch start`'s reconcile in `Command` call `EnsureOrWarn` in place of `Ensure` and no longer return its error.
The Options.Pair call in `battencli`'s wire, which reconciles a pair being created, keeps `Ensure` and its failure, since it is not a read of a stale hub but the creation of a pair whose config must be current.
`CloneAndWire`'s stamp-gated walk keeps `Ensure` and its failure too, because a clone creates the hub rather than reading one, and a walk failure there already leaves the clone intact for the next start verb.

What this lets through, and its bound: a start verb or the Preflight row no longer stops when the reconcile fails, so a run starts on the hub's current, unreconciled config.
Bound: that config is byte for byte what it was before the walk, the stamp stays stale so every later start verb and Preflight row retries the walk, the warning names the file and the way forward, and a module load that still refuses the current config, such as a strict load meeting a retired key, refuses as before, the operator's way forward for a broken file being the Warn printed above the load's refusal in the same output.

`preflightProducer.reconcileHub` calls `EnsureOrWarn` and returns nothing; `preflightProducer.Call` goes on to `preflight.Check` after it, keeping its cancellation check after the reconcile, and `reconcileRefusal`, the Stuck reason only a reconcile failure reached, is deleted with its unit test.
`internal/preflightshed/doc.go`'s paragraph on the reconcile says a failure is logged at Warn with the way forward and the row goes on to the check, no longer Stuck, and no longer that a failure in any pair stops every run in the hub.
`internal/hubreconcile/doc.go`'s Errors section adds `EnsureOrWarn` and the call sites that use it, and `internal/hubgeom/doc.go`'s sentence that the start verbs and the Preflight row hand the geometry to `hubreconcile.Ensure` names `hubreconcile.EnsureOrWarn`.

`TestPreflight_ReconcileFailureWarnsAndGoesOnToTheCheck` replaces `TestPreflight_ReconcileFailureStopsTheRowWithItsWayForward` over the same two fixtures, an unparseable config in another pair and a held hub lock: the row reaches the check and settles Done on the otherwise clean hub, and no stamp is written; the Warn's text is pinned once, in `hubreconcile`'s tests below.
`TestEnsureOrWarn_WarnsWithTheWayForward` is one table-driven top-level test that calls `logcapture.Capture` and never calls `t.Parallel`, as `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` does, and its comment says the logger is process-global, as `PATTERN-test-isolation` asks.
Its rows: a broken hub, where `EnsureOrWarn` returns, the captured Warn names the failing file and `reconcileWayForward`, and no stamp exists, then the fixed file, where a second `EnsureOrWarn` logs no Warn and writes the stamp; and a held lock, where it returns after a Warn naming the lock with no stamp written.
The start verbs' own failure rows turn into go-on rows:
`TestLoomResumeReconcilesTheHubConfigBeforeArming`'s broken-`loom.yaml` step expects `loom resume` to go past the failed reconcile and refuse through the loom config load, which still meets the broken file: exit 1, output naming the file, holding the Warn's `reconcileWayForward` text and the loom config load's own error for that file, and no stamp written; the load's error appearing after the Warn proves the reconcile no longer stopped the verb;
`TestOrchIntegration_Refusals`'s step over a pair's broken `loom.yaml` expects `start` to warn and go on, since orch start loads no loom config: it then meets the `permission_mode: prompt` the earlier "permission mode" step left in the prime's `orch.yaml`, so it still exits 1 and creates no `.lyx/orch`, but its output names `set permission_mode: bypass or remove the key` instead of `lyx config loom`, which proves the reconcile failure no longer stopped it; no stamp exists afterwards, and the step's comment names its reliance on that earlier step.

**ImpactSummary:** one new `hubreconcile` entry point and the four stamp-gated call sites, the start verbs and the Preflight row, move to it, while the clone's walk keeps `Ensure`; the Preflight row's reconcile Stuck reason and its two tests go, the pair call is unchanged.
