# tuned-defaults-template: tuned templates and a sparse hub base

All six batches recorded `done`; the plan-level verify is left to the gate.

## Batches

- **01-loom-tuned-defaults** (cards 1, 2; head `a6f24723`): the loom template takes D1's tuned review models, fans, lenses, budgets, accept and driver; the shed and darn templates take `driver_kind: go` and darn's tuned models and fans.
  Deviations: `internal/loomcli/wiring_test.go`, `internal/loomengine/config_test.go`.
  Departures: none.
- **03-claude-orch-webster-batcher-tuned-defaults** (card 3; head `83651fe1`): the claude, orch, webster and batcher templates take D1's values, and orch's default thresholds follow to 200000 and 150000.
  Deviations: `internal/shuttleengine/claudeengine/prepare_test.go`.
  Departures: none.
- **04-configengine-sparse-hub-base** (cards 4, 5; head `0b2a0da8`): `Spec.SeedOnly`, the sparse base header and board marker, sparse hub `Set` and `Edit`, no fill log for a hub base, the former-values table and the former-value strip.
  Deviations: `internal/configsync/configsync.go`.
  Departures: none.
- **06-sparse-hub-base-reconcile** (cards 6, 7, 8; head `70e19a0c`): `ReconcileHubWideAt` seeds sparse and strips once under `StripBase`, key moves write the base sparse, and the walk strips the base once and commits the marker.
  Deviations: `internal/configsync/configsync.go`, `internal/configsync/configsync_test.go`, `internal/configsync/movekeys.go`, `internal/hubreconcile/ensure_integration_test.go`, `internal/hubreconcile/walk.go`.
  Departures: none recorded as `Pattern-Departure:` trailers; the digest lists none.
- **09-reconcile-failure-warns** (card 9; head `cd08bdd7`): `hubreconcile.EnsureOrWarn` warns with the way forward and the start verbs go on.
  Deviations: none.
  Departures: none.
- **10-config-cli-sparse-hub-base** (cards 10, 11; head `5bf9467b`): `lyx config`'s hub-wide editor scaffolds the sparse header, and `lyx fabric clone` marks a board it seeded from nothing in its own board commit.
  Deviations: `internal/fabriccli/clone.go`.
  Departures: none.

## Scope guard

Each deviation above is a file the scope guard found changed outside its batch's targets; the forks' reports tie them to tests or code the new defaults and the sparse base broke.
