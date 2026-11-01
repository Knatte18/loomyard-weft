# Card 7 — module-bolt-dir-uses

**Edit:**
- `internal/configcli#runReconcile`
- `internal/configcli#resolveReal`
- `internal/fabriccli#hubAnchorDeps`
- `internal/fabriccli#acquireBranchLock`
- `internal/fabriccli#runHubPull`
- `internal/fabriccli#runShortname`
- `internal/fabriccli#runReconcile`
- `internal/reportcli#primeGeometry`
- `internal/hubgeom#CreelGeometry`
- `internal/hubgeom#GateSlots`
- `internal/hubgeom#NearestLiveOrch`
- `internal/hubgeom#ResolveParent`
- `internal/hubgeom#ReconcileGeometry`
- `internal/hubgeom#reedGeometry`
- `internal/hubgeom#burlerGeometry`
- `internal/hubreconcile#Geometry`
- `internal/ideengine#lookFor`
- `internal/orchcli#orchCLI.rulesCmd`
- `internal/orchcli#orchCLI.orchAgentName`
- `internal/orchcli#orchCLI.orchReceiver`
- `internal/orchroute#Paths`
- `internal/orchroute#StrandOf`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#hubOwnerSeams`
- `internal/battencli#lockStore`
- `internal/battencli#ReleaseRunParentLock`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#childParentLock`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#parentLockDeps`
- `internal/loomcli#hubPeersDeps`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#Teardown.awaitRunLockOrPass`
- `internal/tendercli#InstallNetworkGate`
- `internal/tendercli#HubDeps`
- `internal/preflight#BoardLymPresent`
- `internal/gatecli#runLogDirOf`
- `internal/cleanupcli#segmentsOver`
- `cmd/lym#seedStencilsAt`
- `internal/boardengine#OpenHub`
- `internal/testkit/boardkit#HubWithBoardConfig`
- `internal/hubforge#SeedFabricConfig`
- `internal/hubforge#CopyHub`
- `internal/hubforge/doc.go`
- `internal/configcli/doc.go`
- `internal/reportcli/doc.go`
- `internal/hubgeom/doc.go`
- `internal/hubreconcile/doc.go`
- `internal/cleanupcli/doc.go`

**Uses:**
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoardDir`
- `internal/fabriccli/doc.go`
- `internal/orchcli/doc.go`
- `internal/orchroute/doc.go`
- `internal/loomcli/doc.go`
- `internal/pairteardown/doc.go`
- `internal/tendercli/doc.go`
- `internal/preflight/doc.go`
- `internal/gatecli/doc.go`

**Intent:** Move every production path outside fabric that places a thing in the Bolt checkout beside the board onto `fabricengine.BoltDir` (D9), with no behaviour change.

- A `fabricengine.BoardDir(...)` call becomes `fabricengine.BoltDir(...)` where its value reaches anything in the Bolt checkout other than the board's own data: the parent-lock store and the branch lock (`parentlock.Dir`, `acquireBranchLock`), `NewBolt` and the commits, pulls and syncs run in the checkout (`newHubCommit`, `pullBolt`, `boardengine.SpawnSync`, `configsync.ReconcileHubWideAt`), the gate slots and their run logs, hub-wide config directories (`configengine.Dirs{Board: …}`, `{Anchor: …}`, `configengine.ConfigFile`), the checkout's `.lym` (the reconcile stamp and lock, the preflight probe), the shortname record (`ReadShortname`), the hub rules file, the checkout's own git repository in the report geometry, the checkout-existence `os.Stat`s, and the refusal comparing a worktree with the checkout.
- A call whose value reaches only `board.json`, its overview or design files keeps `BoardDir`: `boardengine.ReadOwners`, `SetOwners`, `ReadTasks`, and a board's `Config.Path` or `boardengine.Config{Path: …}`.
  Where one variable feeds both kinds, as in `hubAnchorDeps`, `hubOwnerSeams` and `OpenHub`, it splits into a `boltDir` and a `boardDir`, each from its own function.
- A parameter or field that carries the checkout is renamed to `boltDir` in a private function (`childParentLock`, `acquireBranchLock`) and keeps its name on an exported field (`hubreconcile.Geometry.BoardDir`), whose comment names the Bolt checkout; renaming the exported field is not this task's.
- Every comment in these members that calls such a path the board dir names the Bolt checkout instead, and so do the listed `doc.go` files: `configcli`'s `<BoardDir>/_lym/config/` and board-dir wording for hub-wide config, `reportcli`'s board checkout, `hubgeom`'s gate-slot directory, `hubreconcile`'s stamp, lock and base marker paths, and `cleanupcli`'s logs sweep over the board directory; a `_board` spelling in them is card 5's.
  `boardengine/doc.go` is left to card 13, which rewrites it.
- `hubforge`'s own Bolt-checkout uses of its `Hub.BoardDir` accessor, the repo-wide fabric config `SeedFabricConfig` writes and commits and the binding `CopyHub` commits, take `fabricengine.BoltDir(h.Path)`, and `hubforge/doc.go`'s `SeedFabricConfig` paragraph names the Bolt checkout; the accessor itself stays for the tests that read board files through it.
- Every other production `fabricengine.BoardDir(` call outside `internal/fabricengine` reads or writes the board's own data and stays, and so does `hubgeom.DobbyGeometry`'s fill of `dobbyengine.Geometry.BoardDir`, a field that names the board directory.
- Values are unchanged, since `BoltDir` and `BoardDir` join one name; the suites of each package cover the unchanged behaviour, and no test changes.

**ImpactSummary:** Function-call swaps from `BoardDir` to `BoltDir` across about a dozen packages outside fabric; every path value is unchanged, so no caller or test outcome moves.
