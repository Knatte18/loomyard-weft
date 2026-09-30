---
format: 5
approved: true
language: go
---

# Plan: Trace-log retention — trace-aware, activity-ranked, configurable

Routine lyx invocations evict the trace files a loom run or an ly-drive step depends on:
the sweep counts files rather than traces, ranks by the filename timestamp a long step wrote at its start, and runs at every sink arm.
This plan makes the sweep group files by trace id and rank groups by their newest mtime, moves the sweep to process exit so it runs only in a process whose sink armed, and reads both bounds from a new `_lyx/config/logger.yaml` so the operator can widen the window without a code change.
The config loader lives in a new `internal/loggerconfig` package, since `internal/logger` cannot import `internal/configengine`;
`cmd/lyx`'s exit hook calls it, only after the sink has armed.

## Card Index

1 — retention-trace-groups — sweep by trace group and last activity, with bounds as a parameter
2 — logger-config-module — add `internal/loggerconfig` and `logger.yaml` with the two retention bounds
3 — register-logger-config — register `logger` in configreg so reconcile seeds `logger.yaml`
4 — sink-arm-state — expose whether the sink armed, whether it was redirected, and its directory and anchor
5 — exit-sweep — move the sweep to process exit and feed it the configured bounds

## Shared Decisions

- `RetentionBounds.MaxAge` is a `time.Duration`; `logger.yaml` carries days, and `loggerconfig.Load` converts.
- `logger.DefaultRetentionBounds()` equals the `logger.yaml` template's values, pinned by a card 2 test; a redirected sink and an unusable config both sweep with it.
- The exit hook lives in the CLI-layer root, `cmd/lyx/exitsweep.go` (`notifyExitAndSweep`), and `cmd/lyx`'s `main` and `run` call it in place of `logger.NotifyExit`.
  Of D4's two named sites, cliwire is ruled out because its fixed production dependency set excludes `internal/configengine`.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/logger/... ./internal/loggerconfig/... ./internal/configreg/... ./cmd/lyx/...
```
