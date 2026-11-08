```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each branch and error path the diff changes with a concrete input.

**List (function or branch, case run, verdict)**

- `LoadConfig`, `sandbox_mode` check (`config.go:91`):
  - Template default, no layers: resolves `workspace-write`. Correct.
  - Explicit `sandbox_mode: ""` in a layer: the load fails with `MarkInvalid`, so an empty value fails loudly and does not fall back. Correct.
  - `read-only`: the load fails and the message names the key and value. Correct.
  - Wrong-case value such as `Danger-Full-Access`: refused. Correct.
- `loadConfigOrRefuse`: the error is wrapped with a way forward that now lists `machine`. Correct. `usage.go:21` and `skillload.go:111` call `LoadConfig` directly and propagate the error. Correct.
- `codexCommand` (`command.go:86`):
  - `workspace-write` with and without writable roots: output is identical to before, byte for byte. Correct.
  - `danger-full-access` with writable roots given: gives `-a never -s danger-full-access`, drops the network and writable-root `-c` flags, and keeps `-a never` plus model, effort, fallback and hook flags. Correct.
  - Zero-value `sandboxMode` (empty string): would emit `-s ` with no value. `Prepare` and `OpenLaunch` always set the field from a validated config, so this is unreachable and needs no guard.
- `Prepare` and `OpenLaunch`: both pass `codex.SandboxMode` from the loaded config. Correct. Under `danger-full-access` the writable-roots resolution still runs, so an invalid `writable_roots` still refuses the launch. This is harmless and consistent.
- `LayerFile` machine case (`layers.go:139`):
  - Per-worktree module with a machine key, `Anchor` and `Board` set: the machine layer applies. Correct.
  - Per-worktree module with no machine key: no machine layer. Correct.
  - `Board` alone (a hub-file check): no machine layer. Correct.
  - Standalone load with no `Board`: no machine layer, so `sandbox_mode` comes only from the worktree file or the template. Correct.
- `readLayers` and `stackLayers` order (default, worktree, machine, local): the machine layer sits between worktree and local, and `layerRank` and `BelowLayer` agree. Correct. An empty machine file is skipped. A machine file holding a non-machine key fails `ValidateMachine`. Correct.
- Local override of `sandbox_mode`: `ValidateLocal` refuses it because the key is classified `CategoryMachine`, which is not `CategoryNone`. Correct.
- Other modules' layering: the only per-worktree module that classifies a machine key is codex. Every other module with a machine key (`gate`, `env`, `desktophook`) is hub-wide, so its stack is unchanged. `gateslot/config.go:158` and `:196` call `LayerFile` for hub-wide modules only. Correct.
- `layerPath` and `Set` for the machine layer: the owner is `Board` and the layer is sparse, so nothing is scaffolded. Correct.
- `selectedLayer` (`hubwide.go:115`):
  - Per-worktree module with a machine key: passes through to the machine layer. Correct.
  - Per-worktree module with none (loom): refused naming `--layer local`. Correct.
  - Hub-wide module with none: refused naming the prime's `lym config`. Correct.
  - The per-worktree refusal is checked before the generic one, so the order is right.

No defects found.

**Findings:** none.

**Tool calls (3)**
1. Read `layers.go` 470–620 and `set.go` 15–165: the manifest lacked the stacking, `readLayers` and `Set` machine-layer paths.
2. `grep` for the `loadConfigOrRefuse`, `LoadConfig` and `c.dirs` call sites in `codexengine`: to confirm which callers load the config and with which `Dirs`.
3. The same `grep` also covered the engine constructors in `shuttleengines` and `cmd`: to confirm that `dirs` carries `Board` and `Anchor` into the load.
