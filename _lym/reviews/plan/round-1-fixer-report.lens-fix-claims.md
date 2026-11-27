```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims this fix added or rewrote hold against the code at HEAD.

1. MEDIUM. Card 12 (`12-place-window-child.md`) says a `RolePlacement` record "lives as long as its child, at most the 10-second window wait, the 180-second parent wait and the 120-second lock wait plus its script bounds."
   - The bound leaves out the window command, which the child runs while the record is still registered.
   - Card 12's own `TaskWindow` row asserts the record exists "while the command waits".
   - Card 4 bounds that command by `commandTimeout` (30 seconds) plus `groupKillAfter` (5 seconds).
   - So the record lives up to about 35 seconds longer than stated, and `requireStillOrphan` and the other readers count it for that time too.
   - The fix is to add `afterPlace`'s window-command bound to the sentence.

2. LOW. Card 12's bound says the verb "writes only the told hub's places file … the hub's log files, and the files beside the per-user script lock".
   - Card 8 step 3 also has every script run write the script to a temp file through `lymtemp.CreateTemp`.
   - That file is neither beside the script lock nor in the hub, so the list of what the verb writes is incomplete.

These claims hold:
- **Card 9's heal:** `UpdateJSON` defers `l.Release()`, so it releases `places.lock` as it returns. `ReadJSON` takes `lock.AcquireReadLock` on the same path, and `readJSONUnlocked`'s comment says "acquiring the same lock twice on one path hangs".
- **Card 12's places path:** `<hub>/_bolt/.lym/window-places/` matches `fabriclayout.BoltDir`, which is `filepath.Join(hub, BoltDirName)`, together with card 9's `PlacesPath`.
- **Card 12's reader analysis:** `runproc.List` prunes records of gone pids, so `requireStillOrphan` stops refusing once the child exits. `ownedProcesses`' role checks are `RoleBatten` and `RoleDobby` only. Teardown's `EndSparing` spares only teardown and orch roles, so a placement child gets ended.
- **Card 12's `RefusedMachineMode` row:** `configengine.Set` runs only `ValidateMachine`, which checks leaf paths against the template, so a `sideways` value is written. The refusal comes later from `LoadConfig` (card 3).
- **Cards 13 and 11:** `placewindow_integration_test.go` is an existing `tier1` file (`tiergate.Require(t, 1)`), and `permkit.Chmod` exists at `internal/testkit/permkit`.

Tool calls:
- Read `runproc.List` docs, `fabriclayout.BoltDir`, cancel's End usage and `setEntries`/`validateOverride`, because my context lacked how `List` prunes, the Bolt dir name and whether `Set` validates values.
- Read the `runproc.End*` call sites and `ValidateMachine`, to confirm a placement record is ended at teardown and that a machine-layer `Set` does not refuse an unknown value.
