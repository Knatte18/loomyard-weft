```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set against its surroundings (lens fix-consistency).

**Findings**

1. **LOW** — `placeWindow` doc in `internal/ideengine/spawn.go:113`.
   - The comment still says "for the VS Code window just launched on launchPath", and the parameter is still named `launchPath` (lines 113–117).
   - After the fix, `Spawn` passes `result.windowDir`, which is not the launch path for the prime: it is the hub directory, while the launch path is the workspace file.
   - The comment, the parameter name and the warn log key `"path", launchPath` all describe a launch path, and the value is now the window's working directory. The log line therefore reports the hub dir for a prime spawn, not what was launched.
   - Evidence: the diff changed the `Spawn` call to `placeWindow(l, result.windowDir)`, while `OpenTask` still passes `launchPath`. In `OpenTask` the two are the same value, so the twin was left unrenamed.
   - Fix: rename the parameter to `windowDir`, and say in the comment that it is the directory the command runs in.

2. **LOW** — the file header of `spawn_hub_integration_test.go`, lines 3–5.
   - It says the file "holds the Spawn checks that run against the prime and against a task pair".
   - The fix added `checkWindowCommandFollowsALaunch`, which calls `OpenTask` and exercises a failing `CodeLauncher`, so the header no longer covers what the file holds.
   - The step name in the scenario file ("window command follows a launch and never a failed one") is consistent with the new check.

**Checked and consistent**
- `desktophook`'s package doc and the `AfterWindow` doc still match the new `timing` split.
- The `seedResult` doc matches the `windowDir` field.
- The `Spawn`, `OpenTask` and `spawn.go` file-header comments say only that the window command runs after a launch. They do not name a directory, so the prime change does not contradict them.
- `docs/overview.md:685` says the command runs after the window opens, with no directory stated.
- `internal/desktophook/template.yaml` and the other docs make no claim about the working directory.
- The F7 revert restored the original `configcli` text, so `main` and the branch now agree.
- `export_test.go` has no build tag and `window_test.go` is `tier1 && !windows`, so there is no tag conflict.

**Tool calls**
1. `grep` for "window command|window_command" across docs, Go and yaml files: my context did not show the docs and sibling comments that mention the window command, and I needed them to check for stale directory claims.
