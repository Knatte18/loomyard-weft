```yaml
lens: fix-affected-set
outcome: reported
```

Checked set-premise findings: F1, F4, F7. Every member is covered, with one stale name.

- **F1** (callers passing a file path as the window directory): `placeWindow` has two callers, `internal/ideengine/spawn.go:49` (now `result.windowDir`) and `internal/ideengine/spawn.go:109` (`OpenTask`, a bare-folder `launchPath`, never a file). `desktophook.AfterWindow` has two production callers: `placeWindow` and `internal/reedcli/window.go:39`. The reed caller gets a directory, since the verb checks `IsDir` first. No other site reaches `cmd.Dir` with a workspace file.
- **F4** (launch sites): `CodeLauncher` has two call sites, `Spawn` and `OpenTask` (`internal/ideengine/spawn.go:46` and `:106`). The new test covers both with a failing launcher. `ideengine.Spawn`, `Seed` and `OpenTask` each have one production caller, in `internal/idecli/cli.go`. Nothing is left uncovered.
- **F7** (help-text rewordings): `git diff main` for `internal/configcli/configcli.go` is empty after the revert. Two other rewordings on the branch, in `internal/configengine/doc.go` and `internal/configengine/layers.go`, both describe the machine layer's new desktop keys. Both are in the Goal's scope and the review did not flag them.

Not checked beyond the above: F2, F3, F6 and F8 are single-file test changes with no set premise. F5 is also a single-site test addition.

**LOW** — `internal/ideengine/spawn.go:114-116`, `placeWindow(l, launchPath)`: the parameter name and doc ("window just launched on launchPath") are stale. `Spawn` now passes `windowDir`, which for the prime is the hub directory, not the launch path. Evidence: `spawn.go:49` passes `result.windowDir`. Renaming the parameter to `windowDir` and rewording the doc fixes it.

Tool calls:
1. Grepped non-test Go sources for `AfterWindow`, `placeWindow`, `CodeLauncher`, `vscode.Launch`, `launchPath` and `windowDir`, to enumerate every caller of the changed seams.
2. Diffed the two `configengine` files against main and grepped the callers of `ideengine.Spawn`, `Seed` and `OpenTask`, to judge F7's other rewordings and the F4 caller set.
