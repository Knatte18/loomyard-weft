```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Cases run per function or branch:**

- `envconfig.LoadConfig`
  - No file: returns the template, so `Vars` is an empty non-nil map. Correct.
  - Hub `PORT: 8080`: yaml.v3 decodes it to `"8080"`. Correct.
  - Names `BAD-NAME`, `1ST` and `LYM_X`: refused with `ErrInvalid`. Correct.
  - A list value for a name: the `Unmarshal` error is marked invalid. Correct.
  - The check is case-sensitive, so `lym_strand_name` passes. That is only harmful on Windows, where env names are case-insensitive. Finding 2.
  - `PATH` and similar passthrough names are accepted. Finding 1.
- `Config.Entries` and `Config.Beneath`
  - A nil `Vars` or nil `own` gives an empty non-nil map and nothing is mutated. Correct.
  - `Beneath(nil)` on empty vars turns a nil `step.Env` into an empty map. `renderEnv` only ranges over it, so this is harmless.
- `configengine.templateValueAt` and `scalarEntry`
  - `vars.X` with an empty template map: judged scalar, correct.
  - `vars.X.a`: `rest` is non-empty, falls through to nil, refused as "not in template". Correct.
  - `vars.X: {}`: mapping against scalar, refused for shape. Correct.
  - The local layer refuses via `CategoryMachine`. Correct.
  - Other per-name maps all have template entries, so the `len(entries.Content)==0` branch does not fire for them.
- `yamlengine.Strip` new branch
  - Upper `{}` over lower `{}`: reported as removed. Correct.
  - Upper `{}` over a non-empty lower: still dropped silently, as before. Not in this diff's scope.
- `gatecli.runTest`
  - An env load failure returns `output.Err` before slots, tiers or the build start, so no state is left behind. Correct.
  - Outside a hub, `configDirs` returns `Dirs{Anchor}`, which loads the worktree's own `env.yaml`. Correct.
  - `Entries` are appended after `withBuildServerOff`, so a hub variable can replace a build-server switch lym forces. This looks deliberate and is only documented as replacing inherited values. Finding 3.
  - `LYM_*` names are refused, so `enclosingGateEnv`, `InheritEnv` and `PrebuiltLymEnv`, which are appended or stripped later, cannot be shadowed.
  - Tier and build env are concatenated after, and `os/exec` keeps the last duplicate. Correct.
  - `green.go` `expandPackages` receives `e.env`, so `go list` sees the variables too. Correct.
  - The variables stay out of the green-record fingerprint, as documented. A changed env value therefore does not invalidate a carried green tier. This matches the stated design, noted only.
- `driftengine.Config.StepWithHubEnv`
  - An undeclared step returns the `Step` error. Correct.
  - An env load failure returns an error and no step. Correct.
  - A step value beats the hub value, and the config's own `Env` is not mutated because `Beneath` clones. Correct.
  - `driftcli` `takeRun` and `runCaptureChild`, and `loomcli` `driftDeps`, all use it. Both capture and compare get the variables. The remaining `cfg.Step` calls in `driftcli/cli.go:322`, `driftengine/baseline.go:185` and `verify.go:173` only validate that a step is declared.
- `hubgeom.paneEnv` and `reedengine.paneEnvExports`
  - Nil `PaneEnv`: exports nothing. Correct.
  - A read error: logs a warning naming the strand and exports nothing. The launch continues, and `paneEnv` returns `cfg.Vars` as nil beside the error, which is harmless.
  - Quoting: POSIX `Quote` single-quotes the value, and names are validated by `namePattern`, so no injection. Correct.
  - `launchCmd` is a local variable reassigned at `spawn.go:119` and not reused after, so the exports are not persisted into state. Correct.
  - The exports run after the prelude, which prepends lym's directory to `PATH`. Finding 1.
  - Detached windows from `OpenWindow` do not export the variables. The decision record names agent sessions only, so I note it and do not flag it.

**Findings**

1. **MEDIUM, `design`.** `internal/envconfig/config.go:283-287` with `internal/reedengine/spawn.go:119-120`.
   - `vars.PATH` passes validation. Reed emits the exports after the prelude that prepends the spawning `lym` directory to `PATH`, so `export PATH='...'` replaces it.
   - That breaks the documented property that every strand pane resolves `lym` to its spawning binary (`reedengine/doc.go:211`).
   - Only `LYM_` is reserved, so `PATH`, `HOME`, `SHELL` and `TMUX*` are all writable.
   - The fix is to refuse the names the prelude relies on, or to place the exports before the prelude.
2. **LOW, `design`.** `internal/envconfig/config.go:284`.
   - The `LYM_` check is case-sensitive. On Windows, where lym runs with the pwsh dialect, `Lym_Strand_Name` or `lym_parent` shadows lym's own variables.
   - The fix is to compare with `strings.HasPrefix(strings.ToUpper(name), reservedPrefix)`.
3. **LOW, `design`.** `internal/gatecli/run.go:138`.
   - The env module's entries are appended after `withBuildServerOff`, so a hub variable can undo the build-server-off switches lym forces for every run, such as `MSBUILDDISABLENODEREUSE`.
   - The docs say the module replaces inherited values only. The fix is to append the entries before `withBuildServerOff`.

**Tool calls**
1. Read `reedengine/spawn.go:100-170`: the manifest did not show whether `launchCmd` is reused after reassignment, or its ordering against the prelude.
2. grep over `configengine` for `Set` and its base-layer validation: to check whether `lym config env --set vars.NAME=value` works on the hub layer. It only located `set.go:28` (the `LayerDefault` and `LayerMachine` branch). I did not read it, so hub-layer `Set` is unverified. Note: no test pins hub-layer `Set`, only machine.
