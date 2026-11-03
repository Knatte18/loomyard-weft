# Add `lym config env`: per-machine variables for every lym process

A hub could not hold an environment variable whose value differs per machine, such as a path to a sibling checkout.
The only place lym set a variable was a gate tier's `tiers.<n>.env`, and the machine layer refuses `tiers` as an executable key.
A per-machine value had to be committed into the shared hub file, where it is wrong on every other machine.

This adds an `env` config module, `lym config env`, that holds a `vars` map from variable name to value.
The hub file in the board dir holds the shared values, and the never-tracked machine file overrides them name by name.
`vars` is classified as a machine key, so the machine layer accepts it and the local layer refuses it.

lym exports the variables to every process it starts for the hub:

- gate tiers and the shared build (`internal/gatecli`);
- Drift capture and compare steps (`driftengine.Config.StepWithHubEnv`, wired in `internal/driftcli` and `internal/loomcli`);
- agent session panes, through a lazy `PaneEnv` closure on the reed geometry (`internal/hubgeom`, `internal/reedengine`), exported ahead of the strand's launch command and read afresh at each launch.

Precedence, most specific first: a gate tier's or build's own `env` (or a Drift step's), then the `env` module, then the inherited environment.

Closes #19

## Design decisions

- `envconfig.LoadConfig` refuses a name that is no shell-exportable variable name, `PATH`, and any name starting `LYM_`, in any letter case.
  `PATH` carries the directory a strand pane's prelude puts first so `lym` resolves to its spawning binary, and `LYM_` names are lym's own.
- The .NET build-server switches in `proc.BuildServerOffEnv` stay lym's own: they are appended after the module's entries in gate runs, and dropped from the module's exports in strand panes.
- A hub with no `env.yaml` resolves the template, which sets nothing, so an unreconciled hub is unaffected.
- A failed env read in a strand launch logs a warning and exports nothing, so the strand still launches.
  A failed read in `gate test` is a refusal.
- `configengine` judges a name under a per-name map that has no template entry as a scalar, so `--set vars.NAME=value` validates in the machine layer.
- `yamlengine.Strip` treats an empty mapping over an empty lower mapping as a removed unit, so a machine file that restates an empty `vars: {}` is stripped.
- `PATTERN-config-layers.md` and `docs/overview.md` list the new module.

## To check by hand

- `drift-wiring-sites-unpinned`: the Drift wiring sites in `internal/driftcli` and `internal/loomcli` that pass the hub env layers are only partly pinned by tests.
- `gate-hub-config-dirs-untested`: the hub config directories `internal/gatecli` hands the env module, `configDirs` in `run.go`, are only partly covered by tests.
- `reed-launch-export-order-untested`: the order of the env exports ahead of the launch command in `internal/reedengine/spawn.go` is only partly covered by tests.
- `shared-build-env-untested`: the shared build's receipt of the module's variables in `internal/gatecli` is only partly covered by tests.
- Set a variable with `lym config env --layer machine --set vars.NAME=value` and confirm it reaches a gate tier, a Drift step and an agent pane.
