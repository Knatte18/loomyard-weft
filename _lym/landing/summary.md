# Per-machine environment variables: the `env` config module

A hub could set an environment variable for a run only through a gate tier's `tiers.<n>.env`, in the hub file committed in _bolt.
The machine layer refuses `tiers`, since it is executable, so a per-machine value such as `COMMONLIB_REPO`, a path to a sibling checkout, had to be committed hub-wide, where it is wrong on every other machine.

## What changes

- **New hub-wide config module `env`** (`internal/envconfig`).
  Its one key, `vars`, maps a variable name to a value.
  It is a per-name map classified `machine`, so:
  - the hub file holds values shared by every machine: `lym config env --set vars.NAME=value` from the prime;
  - the never-tracked machine file holds this machine's own: `lym config env --layer machine --set vars.NAME=value`;
  - the machine value wins name by name, and the hub's other names stay;
  - the local layer refuses the key.

  `LoadConfig` degrades to the template, which sets nothing, when the hub holds no `env.yaml`.
  It refuses a name that is not a valid environment variable name, and one starting `LYM_`, so the module never overrides lym's own variables.
- **Exported to every process lym starts for the hub**, with precedence, most specific first: a tier's, build's or Drift step's own `env`, then the `env` module, then the inherited environment.
  - Gate (`internal/gatecli/run.go`): the variables are appended to the base environment after the build-server switches, so they replace inherited values, and the tier's or build's `env`, appended later, replaces them.
    Outside every hub the worktree's own `env.yaml` applies, as the gate config does.
  - Drift: the new `driftengine.Config.StepWithHubEnv` lays the variables beneath a step's own `env`.
    Every drift verb (`driftcli`) and Drift row (`loomcli/driftwiring.go`) that runs a command now takes its step through it.
  - Agent sessions: `reedengine.Geometry` gains `PaneEnv`, a lazy closure that `hubgeom` fills with the env module loaded under the hub's dirs.
    `launchStrandLocked` exports the variables, in name order, ahead of the strand's launch command at every launch and relaunch, so an agent that runs tests by hand sees the same values.
    A failed read logs a warning and launches the pane without them.
- **configengine**: under a per-name map with no template entry, the local and machine layers now judge a name as a scalar, the shape yamlengine's set already writes there.
  Before, every name under such a map was refused as "not in the template", which would have refused every machine write to `vars`.
  Each existing per-name map has template entries, so their behavior is unchanged.
- **yamlengine**: `Strip` now reports an empty mapping that equals the template's empty mapping as a removed unit.
  Before, it dropped that mapping silently, and with nothing reported a hub's one-time base strip left an untuned `env.yaml` holding the whole template.
  Every other module's empty template mapping is an open map, which `Strip` already compares whole.

## Design decisions

- `vars` is a per-name map, not an open map, because an open map from a higher layer replaces the lower one whole: a machine file setting one variable would have dropped every hub variable.
- The gate adds the module's variables to the base environment, not to each tier's `env`, so they stay out of the tier fingerprint, as inherited variables do.
- Reed puts the exports ahead of the launch command, after its own `LYM_STRAND_NAME` and `LYM_PARENT` exports.
  Refusing `LYM_` names keeps the module from overriding them.
- The `machine` category's definition widens from "a fact of the machine's desktop" to "a fact of the machine, such as its desktop or a path on its disk", in `configengine` and `pattern/PATTERN-config-layers.md`.
  `envconfig` joins that background file's list of degrading callers.

## Tests

- `internal/envconfig/config_test.go`: the hub-plus-machine stack merged name by name, number values read as text, refused names (`-`, a leading digit, `LYM_`), a refused local layer, and a refused mapping value.
  It also pins the issue's refused write: `Set` on the machine layer for `vars.COMMONLIB_REPO` succeeds, and the same write on the local layer is refused and leaves no file.
- `internal/driftengine/config_test.go`: `StepWithHubEnv` keeps the step's own value over the module's and leaves the config's step unchanged.
- `internal/reedengine/panebin_test.go`: `paneEnvExports` per shell dialect, for no seam, an empty answer, sorted exports, and a failed read.
- `internal/gatecli/test_integration_test.go` (tier 1): a tier sees the module's `BUILD_MARK` over an inherited one, and keeps its own `GOFLAGS` over the module's.
- `internal/configreg/configreg_test.go`: `env` added to the hub-wide modules, with `vars` as its machine key.
- `internal/yamlengine/layers_test.go`: a `Strip` row for an empty mapping equal to an empty lower.
- `cmd/lym/configstrictness_test.go`: `internal/envconfig` added to the pinned callers that fall back to the template.
