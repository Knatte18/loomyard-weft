# Codex engine: a sandbox_mode machine key picks the launch's sandbox

Every Codex launch ran `-a never -s workspace-write`, hardcoded on the launch line.
On a machine whose bubblewrap may not create a user namespace, such as Ubuntu with `kernel.apparmor_restrict_unprivileged_userns = 1` and no AppArmor profile for bwrap, that sandbox cannot start.
Every shell command then fails with `bwrap: setting up uid map: Permission denied`, and the session stalls or exits without writing its output.
No configuration could change the mode: `-s` overrides a project `.codex/config.toml`, and `binary`, the one escape a wrapper script used, may not be set in a machine or local layer.
This change resolves issue #83.

## What changes

`codex.yaml` gains `sandbox_mode`, `workspace-write` by default.
`danger-full-access` runs the session with no sandbox.
The load refuses any other value, and the launch's refusal names the way forward through `lym config codex`.
Under `danger-full-access` the launch line drops `sandbox_workspace_write.network_access` and `sandbox_workspace_write.writable_roots`, since there is no sandbox for them to configure.
The workspace-write line is unchanged byte for byte.

`sandbox_mode` is classified `CategoryMachine`.
Whether bubblewrap can start Codex's sandbox is a fact of the machine, so one machine can run unsandboxed while the tracked files keep the sandbox on every other machine.
The local override refuses the key, so no worktree's agent can lift the sandbox from its own untracked file.
To switch a machine, run this from the hub's prime worktree:
`lym config codex --layer machine --set sandbox_mode=danger-full-access`.

## Design decisions

- **The machine layer opens to per-worktree modules.**
  Until now only a hub-wide module stacked `<Bolt>/.lym/config/<module>.machine.yaml`, and `codex.yaml` is per-worktree.
  `configengine.LayerFile` now applies the machine layer to any module with a `CategoryMachine` key, given both `Anchor` and `Board`.
  A per-worktree module's stack becomes template, repo default, worktree, machine, local, so a machine value wins over every tracked file.
  `lym config --layer machine` no longer refuses a per-worktree module that has machine keys.
  A module with no machine key is still refused, and a per-worktree module's refusal still names `--layer local`.
  The configreg test that refused a per-worktree module with a machine key is removed, since such a key now has a loader.
  No other per-worktree module classifies a machine key, so no other stack changes.
  `pattern/PATTERN-config-layers.md`'s machine-layer section records the new reach.
- **No bwrap preflight.**
  The issue asked for the key, a preflight or both.
  The key alone lets an affected machine run, and a preflight would start a bwrap child process on every launch to diagnose a fault that a machine has or lacks permanently.
- **Keys left as they were.**
  `binary` stays `CategoryExecutable` and `writable_roots` stays `CategoryGuard`.
  The wrapper workaround is no longer needed, but nothing here removes it from a hub that set one.

## Tests

- `TestBuildLaunchCmd` gains a danger-full-access row with writable roots given, which the line leaves out.
- `TestPrepare_Launch` gains a row where the worktree layer sets danger-full-access.
  `TestPrepare_RefusesBeforeWriting` refuses an unknown mode with the way forward.
- `TestLoadConfig_SandboxModeLayers` pins that the machine override wins over the worktree file in a hub, and that a local override setting the key refuses the load.
- `TestLayerFile` pins the machine layer for a per-worktree module with a machine key, and none for one without.
- `TestDispatchLayers` writes `codex`'s machine key uncommitted from the prime.
- `TestModules_EveryTemplateKeyClassified` records `sandbox_mode` as codex's machine key and among its local-denied keys.
