```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1** (`pattern/PATTERN-lym-files.md:25`): the shortened line holds. `MachineFile(board, module)` in `internal/configengine/layers.go:117` always resolves under the board dir's `.lym/config/`. `PATTERN-config-layers.md:44` also says the layer applies to any module with a machine key, hub-wide or per-worktree.
- **F2** (`docs/overview.md:513`): the reworded sentence matches `configengine/doc.go:54`, and the following path line (`<BoltDir>/.lym/config/<module>.machine.yaml`) is still correct for codex.
- **F3** (`open_test.go`): the `FullAccessSandboxMode` row would fail if `OpenLaunch` hardcoded workspace-write. The expected line carries `-s danger-full-access` and the hardcoded one would carry `-s workspace-write`, so `DeepEqual` differs. Expecting the same `writableRoots` is harmless, because `command.go:87` drops them under full access.

Tool calls:
1. Grepped `configengine` for the machine-file path helpers and listed the files commit 45aad0b1 touched. My context lacked where the machine file resolves.
2. Grepped for "hub-wide" wording in `pattern`, `docs` and `internal`. My context lacked other stale "hub-wide machine" sentences.
