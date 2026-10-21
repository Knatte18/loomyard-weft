# Gate settings per machine: machine layer, gate keys, CLI

All three batches recorded `status: done`, with no `departures`.

- **01-config-machine-layer** (head `42b798c6`): added the machine layer to `configengine`.
  The scope guard listed `internal/configengine/layers.go`, `internal/configengine/layers_test.go` and `internal/configengine/set.go` as deviations.
- **02-gate-machine-keys** (head `bdb1d520`): classified gate's `slots` and `go_parallel` as machine keys and reworded the gate load failures.
  The scope guard listed `internal/gateslot/config_test.go` as a deviation.
- **03-config-cli-machine-layer** (head `619de77d`): added `lyx config --layer machine`, and pinned that `hubreconcile.Ensure` leaves a machine file unchanged.
  The scope guard found no deviations.

The plan-level verify gate runs after this turn ends.
