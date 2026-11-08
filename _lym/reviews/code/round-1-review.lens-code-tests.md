```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it.

**Per behaviour**
- danger-full-access launch line drops network access and writable roots: `TestBuildLaunchCmd/DangerFullAccess` (`command_test.go`). It passes writable roots and expects them omitted.
- workspace-write line unchanged byte for byte: `TestBuildLaunchCmd/EveryInput`, `TestBuildLaunchCmd/NoWritableRootModelOrEffort` and the opened-session assertion in `TestBuildLaunchCmd`.
- `Prepare` passes `Config.SandboxMode` to the launch line: `TestPrepare_Launch/DangerFullAccess`, where the worktree layer sets the mode.
- `OpenLaunch` passes the mode through: `TestOpenLaunch` (`open_test.go`), workspace-write only. It would fail if the field were dropped. See finding 1.
- Template default is workspace-write: `TestLoadConfig_Template`. The template.yaml line is covered by that default.
- An invalid `sandbox_mode` refuses the load, and the refusal names the `machine` layer as a way forward: `TestPrepare_RefusesBeforeWriting/UnknownSandboxMode`. An empty value takes the same branch, so a separate row would be redundant.
- The machine layer wins over the worktree file for a per-worktree module: `TestLoadConfig_SandboxModeLayers/MachineOverWorktree`.
- A local override of a machine key is refused: `TestLoadConfig_SandboxModeLayers/LocalRefused`.
- `codex`'s `sandbox_mode` is classified machine and local-denied: `TestModules_EveryTemplateKeyClassified`, through the `wantMachineKeys` entry and the `LocalDenied` list.
- `LayerFile` applies the machine layer to a per-worktree module with a machine key, and not to one without: `TestLayerFile`, the two new and changed rows.
- Other modules' layering is unchanged: the `TestLayerFile` rows for no machine key, hub-wide or per-worktree, and the existing rows for board-alone and anchor-alone.
- `lym config --layer machine` on a per-worktree module with a machine key writes the file uncommitted: `TestDispatchLayers`, the new `codex` row.
- `selectedLayer` still refuses a per-worktree module with no machine key and names `--layer local`: `TestDispatchLayers`, the renamed `loom` row.
- The hub-wide no-machine-key refusal is unchanged: existing `TestDispatchLayers` rows. This is not visible in the diff, so I did not check it.
- The configreg rewrite lost no coverage. The removed per-worktree-with-machine-key refusal test and `machineKeyRefusal` enforced a rule the diff reverses. The `MachineKeys` check per module remains.

**Findings**
1. LOW, scope: no test sends `danger-full-access` through `OpenLaunch`.
   - Location: `internal/shuttleengine/codexengine/open_test.go:81`.
   - Evidence: the only `OpenLaunch` expectation is `sandboxMode: sandboxWorkspaceWrite`, and the engine reads `codex.SandboxMode` at `codexengine.go` in `OpenLaunch`.
   - A hardcoded `sandboxWorkspaceWrite` in `OpenLaunch` would pass every test. An opened session on a machine without a working bubblewrap would then still stall.
   - Fix: add a row to `TestOpenLaunch` whose worktree layer sets `danger-full-access`.
2. LOW, scope: a machine-layer set of an invalid `sandbox_mode` on a per-worktree module has no test.
   - Location: `internal/configcli/configcli_test.go:1566`.
   - The new `codex` row writes a valid value only. The "value the loader refuses over the prime's worktree stack leaves the file as it was" path is pinned only for `gate` (row near line 1560). That path now also guards `codex`'s load check, which includes the new mode validation.
   - Fix: add a `TestDispatchLayers` row for `codex` with `sandbox_mode=read-only`, expecting a non-zero exit and no machine file.

No new test duplicates an existing one:
- `WorktreeWithNoMachineFile` runs the board-plus-anchor path, which the anchor-only `Prepare` row does not.
- The `TestLayerFile` rows are distinct.

Tool calls: none; the inherited context held every diff and test I needed.
