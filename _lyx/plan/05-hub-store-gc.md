# Card 5 — Hub stores never auto-gc; lyx runs gc at pair removal

**Create:**
- `plan:internal/fabricengine#gcAutoThreshold` -> `var gcAutoThreshold = 6700`
- `plan:internal/fabricengine#disableStoreAutoGC` -> `func disableStoreAutoGC(l *lyxcwd.Location) error`
- `plan:internal/fabricengine#housekeepStores` -> `func housekeepStores(l *lyxcwd.Location)`
- `plan:internal/fabricengine#SetGCAutoThresholdForTest` -> `func SetGCAutoThresholdForTest(t *testing.T, threshold int)`
- `internal/fabricengine/housekeeping_integration_test.go`
- `pattern/PATTERN-hub-store-gc.md`

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine/doc.go`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#RecordsRepoRoot`

**Intent:** Decision D2: the hub's shared code and records object stores hold more unreferenced loose objects than git's auto-gc threshold, so nearly every commit, merge or fetch, agents' and the operator's included, starts a detached repack under live steps.
Turn git's automatic gc and maintenance off in both stores, and run gc in the foreground where lyx is already doing slow work: after a pair's removal.

New file `internal/fabricengine/housekeeping.go`, deliberately not `destroy.go`: it removes no worktree, branch or file the destruction gate protects, and what gc deletes is unreachable objects and expired reflog entries past git's own default expiry.
It must contain none of `cmd/lyx/destructiveguard_test.go`'s banned tokens.

- `disableStoreAutoGC` sets `gc.auto=0` and `maintenance.auto=false` with `git config` in the shared config of the code store (through `l.WorktreePath()`) and of the records store (through `RecordsRepoRoot(l)`), via `gitexec.Run` (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`).
  `maintenance.auto=false` turns off git's post-command `git maintenance run --auto`, whose repack tasks under a `geometric` or `incremental` `maintenance.strategy` `gc.auto` does not govern.
- `housekeepStores` calls `disableStoreAutoGC`, then runs `git -c gc.auto=<gcAutoThreshold> -c gc.autoDetach=false -c gc.worktreePruneExpire=never gc --auto --quiet` in each store, so it runs in the foreground, only when the store is above git's default threshold, and never prunes a worktree's admin entry, which stays under fabric's destruction gate.
  It is best-effort and returns nothing: each failure, including another gc holding `gc.pid`, is logged as a warning through `internal/logger` and never fails the caller; each gc run logs its spawn and its end (`PATTERN-spawn-observability`).
- `gcAutoThreshold` is git's default `gc.auto`; it is a variable only so a test can lower it through `SetGCAutoThresholdForTest`, in `export_test.go`, restoring it in `t.Cleanup`.
  It is not operator configuration.

`Topology.Add` calls `disableStoreAutoGC` once its pre-flight refusals have passed and before its first mutation; a failure is logged as a warning and never fails the Add.
`Topology.Remove` calls `housekeepStores` after a successful teardown, after both worktrees are gone and before it returns its result, so the `PATTERN-pair-teardown` order is untouched; both `lyx fabric remove` and batten's Worktree-Teardown reach it through `internal/pairteardown`.
A refused or failed Remove runs no housekeeping.
Both methods' doc comments state the step.
An existing hub picks the keys up at its next pair create or removal.

Bound, stated in `doc.go` and the PATTERN background: the two keys disable git's automatic gc and maintenance for everyone using the store, the operator's manual git included; lyx's housekeeping is the replacement, a manual `git gc` still works, and maintenance an operator registers with `git maintenance start` stays outside lyx's control.

`internal/fabricengine/doc.go` describes the store settings and the housekeeping step.
PATTERN: add one short entry under `## Fabric and git`, `PATTERN-hub-store-gc` — hub stores carry `gc.auto=0` and `maintenance.auto=false`, and fabricengine's housekeeping at pair removal is the only lyx code that runs `gc` or writes `gc.*` or `maintenance.*` on them.
Its background file `pattern/PATTERN-hub-store-gc.md` holds the carve-out: housekeeping deletes only unreachable objects and expired reflog entries past git's default expiry, never a worktree's admin entry, and is not a destructive primitive under `PATTERN-fabric-destruction-chokepoint`; the operator's manual `git gc` and `git maintenance start` stay outside lyx's control.
The entry fits in the room card 2 freed under `MaxOverviewBytes`.

Tests: `housekeeping_integration_test.go` holds one integration test over hubs built with `hubforge.NewHub` and `hubforge.AddPairWith` (`PATTERN-hubforge-fixtures`), not parallel because it lowers `gcAutoThreshold` and captures `logger.SetOutput` (process-global state, named in a comment).
hubforge already sets both keys, so each subtest first unsets them in both stores with `git config --unset`.
Its subtests:

- After a pair is created, both stores' config holds `gc.auto=0` and `maintenance.auto=false`.
- After a pair is removed, both stores' config holds both keys again.
- With the threshold lowered and reachable loose objects committed in a store so that its sampled `objects/17` directory is above it, a Remove leaves those objects packed (absent from `objects/17`) before it returns.
- With `gc.pid` held in a store by a live process on this host, a Remove still succeeds and the captured log holds a warning naming the gc failure.

Raise `internal/fabricengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** Every pair create writes two config keys into both hub stores, and every successful pair removal runs a foreground `gc --auto` in both, best-effort; the destruction gate, teardown order and every result type are unchanged.

**Commit:** 5: fabric: hub stores never auto-gc; gc runs at pair removal
