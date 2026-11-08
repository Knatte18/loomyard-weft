# Codex: add a machine-settable sandbox_mode key

Every Codex launch from `lym shuttle` passed `-a never -s workspace-write`, hardcoded in `codexengine/command.go`.
On a machine whose bubblewrap cannot create a user namespace (Ubuntu with `kernel.apparmor_restrict_unprivileged_userns = 1` and no AppArmor profile for bwrap),
every shell command in the session failed with `bwrap: setting up uid map: Permission denied`.
The run then stalled or died and wrote no output, and no configuration could change the mode.

`codex.yaml` now has a `sandbox_mode` key, `workspace-write` by default, that may also be `danger-full-access`.
It is set per machine with `lym config codex --layer machine` from the hub's prime worktree.
Any other value refuses the config load.

Closes #83

## Design decisions

- **Machine key, not local.**
  Whether bubblewrap can start is a fact of the machine, so `sandbox_mode` is `CategoryMachine`.
  A local override may not set it, since it lifts the sandbox.
- **Machine layer now serves per-worktree modules.**
  `codex` is a per-worktree module, and the machine layer previously applied only to hub-wide ones.
  `configengine.LayerFile` and `configcli` (`--layer machine` selection, help text, refusals) now accept any module with a machine key.
  The machine file stacks above the worktree file and below the local file.
  A per-worktree module with no machine key still refuses `--layer machine`, naming `--layer local`.
- **`danger-full-access` launches carry no sandbox flags.**
  The launch line omits the sandbox network access setting and the writable roots, which only apply under `workspace-write`.
  The same holds for `OpenLaunch`.
- **No preflight.**
  The issue asked for the key, the bwrap preflight, or both; only the key is implemented.

Docs (`docs/overview.md`, `pattern/PATTERN-config-layers.md`, `pattern/PATTERN-lym-files.md`, package docs) are updated to match.

## To check by hand

- Not run: a real Codex session on a machine where bwrap fails, with `sandbox_mode: danger-full-access` set in the machine layer.
  Confirm shell commands run and the output file is written.
