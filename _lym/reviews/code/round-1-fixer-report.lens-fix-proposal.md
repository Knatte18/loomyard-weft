```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Each reviewer proposal I challenged held up:
- **F1** (`filepath.Dir(workspacePath)` as the prime's window directory): `placeWindow` takes its config from `ConfigDirs(l)`, not from `dir`, so the hub directory only has to exist. The new prime assertion in `checkPrimeWritesHubWorkspace` would fail with the defect in place, because `cmd.Dir` set to a file fails the start with ENOTDIR.
- **F2/F3** (injectable wait and timeout): `AfterWindow` still passes `productionTiming` (30s, 5s), so production behavior is unchanged. In `TestAfterWindow`:
  - Removing the sleep fails `wantSleeps`.
  - The `exec sleep 60` row fails the `elapsed > 10s` check, or returns nil, if the bound is removed.
  - The `markerPresentAtSleep` check pins command-after-wait ordering.
- **F4** (`OpenTask` and failing launcher): `recordLauncher` captures the original launcher first, so reassigning `CodeLauncher` afterwards is still restored by its cleanup. `resetPrimeEditorState` and `AddPair` give `Spawn` and `OpenTask` a valid task slug.
- **F5** (`placeTerminalWindow` test): the parent's `t.TempDir()` outlives its parallel subtests. The marker is written only into the `CopyHub` copy of the hub, not into any shared fixture.
- **F6/F8** (blank and unset rows): asserting that no wait ran, with a nonzero `window_wait_ms: 7`, catches removal of the `TrimSpace` early return. A marker check alone could not, since a blank command in `sh` writes nothing.
- **F7** (revert): the same wording change has no twin elsewhere. The `git diff main` hunks in `internal/configengine/doc.go`, `internal/configengine/layers.go` and `pattern/PATTERN-config-layers.md` only extend the machine-layer category text and add `desktophook` to the degrading-callers set. Neither touches the "Keys the current template does not recognize are" sentence.

One weak point, below LOW: the hang row's `wantErr: "window command"` matches every `runWindowCommand` failure. The `elapsed > 10s` check is what pins the timeout, so the weakness is harmless.

Tool calls (1 of 6):
1. `git diff main` on `internal/configengine/doc.go`, `internal/configengine/layers.go` and `pattern/PATTERN-config-layers.md`: to check whether the `configcli` rewording had twins or dependents. My context did not hold those files.
