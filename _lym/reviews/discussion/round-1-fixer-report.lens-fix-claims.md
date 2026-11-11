```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

No findings.

What I checked:
- **D8 migration order.** `ReconcileHubWideAt` runs `m.Migrate` (configsync.go:1140) before `pruneSparse` (1157), and the walk calls `applyReinterpretations` after it (walk.go:254). `pruneSparse` drops the units a retired key covers even without `PruneUnknown` (configsync.go:719-739). The claim holds.
- **D8 claim that the migration also covers beam's other layers.** The claim is that the same migration runs on the prime's `beam.yaml` and on beam's local override file. `ReconcileAll` calls `reconcileSparseFile` for the worktree layer and the local layer (configsync.go:524, 532), and that function runs `m.Migrate` (674). The hub walk calls `ReconcileAll` per worktree (walk.go:341). The claim and its D8 criterion hold.
- **D8 hand-start rule.** The claim is that the dobby starts no pass while a live `RoleTestMap` record stands. The existing `testMapRunning` check does this (dobbydeps.go:588). The claim holds.
- **D11.** `dobbyPassInFlight` counts any live child of a dobby PID that is not push or sync. The refresh is a direct child of the dobby (`startTestMap`, which keeps `cmd.Wait`), so today it is counted, and taking it out of the count is a sound change. For the prime, `dobbyOwner` returns `runproc.HubOwner` (anchordeps.go:92-96), where the refresh is registered (dobbydeps.go:619). `EndDobbies` sees `RolePush` and `RoleSync` only (anchor.go:468), so `EndSparing` ends `RoleTestMap`. The claim holds.
- **D10.** The claim that `t.Parallel()` comes after `tiergate.Require` matches pool_test.go:40-42 and coveragemap_test.go:16-17.
- **D6.** The `env.yaml` digest matches `gateEnv(r.request.environ, hubEnv)` (testmap.go:265).
- **D9 bound** ("ends within one pass job"). A job either releases its lock when it ends or gives it back on a yield. The claim holds.

Calls made:
1. configsync.go:505-540, anchordeps.go:459-510 and 240-258. My context lacked `ReconcileAll`'s layer loop and the bodies of the in-flight check and `EndDobbies`.
2. `dobbyOwner`, the `EndDobbies` call site, the `startTestMap` registration and the `RoleTestMap` users. My context lacked which owner the prime's dobby and the refresh register under.
3. `dobbyOwner` and `HubOwner` definitions, `EndSparing`. My context lacked the spare semantics.
4. anchordeps.go:90-100 and anchor.go:462-470. My context lacked the `dobbyOwner` body and the roles passed to `EndDobbies`.
5. `ReconcileAll` callers. My context lacked whether the hub walk reconciles worktree layers.
