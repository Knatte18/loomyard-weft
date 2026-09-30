# Trace-log retention: trace-aware, activity-ranked, configurable

All five batches finished with `status: done`, and the plan-level verify (`go build`, `go test`, integration-tagged tests) reported OK at head `1539c571f`.

- **01 retention-trace-groups** (`261ceffe9`): the sweep in `internal/logger` groups files by trace id and ranks groups by newest mtime.
  The bounds are now a parameter.
- **02 logger-config-module** (`2aa965252`): new `internal/loggerconfig` package with the `logger.yaml` template, `Load` and `ConfigPath`.
  Also touched `cmd/lyx/configstrictness_test.go`, `CONSTRAINTS.md` and `docs/shared-libs/README.md`.
- **03 register-logger-config** (`935d37b4c`): `logger` registered in `configreg`, with a reconcile-seeding test.
  Deviation: `internal/configreg/configreg.go` and `internal/loggerconfig/template.yaml`.
  The template's inline-comment spacing was changed to match what reconcile writes.
- **04 sink-arm-state** (`230268330`): `internal/logger` exposes whether the sink armed, whether it was redirected, and its directory and anchor.
- **05 exit-sweep** (`1539c571f`): `cmd/lyx/exitsweep.go` (`notifyExitAndSweep`) runs the sweep at process exit with the configured bounds, called from `main` and `run`.
  The sweep was removed from sink arming.
