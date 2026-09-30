# Card 2 — logger.yaml config module

**Edit:**
- `cmd/lyx#configStrictnessDegradingSet`
- `CONSTRAINTS.md`
- `docs/shared-libs/README.md`

**Create:**
- `internal/loggerconfig/template.yaml`
- `internal/loggerconfig/template.go`
- `internal/loggerconfig/config.go`
- `internal/loggerconfig/config_test.go`
- `plan:internal/loggerconfig#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/loggerconfig#Load` -> `func Load(anchorPath string) (logger.RetentionBounds, error)`

**Uses:**
- `plan:internal/logger#RetentionBounds`
- `plan:internal/logger#DefaultRetentionBounds`
- `internal/configengine#LoadOrTemplate`
- `internal/batcher#Active`

**Intent:**
Add `internal/loggerconfig`, the module owning `_lyx/config/logger.yaml` (decision record D4, D5).
It lives outside `internal/logger` because `internal/configengine` imports `internal/logger`, so the logger cannot load its own config without a cycle.
Follow `internal/batcher`'s config shape (`template.yaml` embedded by `template.go`, a `config.go` calling `configengine.LoadOrTemplate`).

- `template.yaml` carries `trace_retention_count: 200` and `trace_retention_days: 14`, each with a trailing comment stating what it bounds, in the style of the other module templates.
- `template.go` embeds it and exposes `ConfigTemplate()`.
- `config.go` declares the package doc, a `moduleName = "logger"` constant, and `Load(anchorPath)`:
  it calls `configengine.LoadOrTemplate(anchorPath, moduleName, []byte(ConfigTemplate()))`, unmarshals both keys into `int` fields, and returns an error naming the key when either value is zero or negative.
  A non-integer value (a float, a string) fails the unmarshal and returns that error.
  On success it returns `logger.RetentionBounds{Count: count, MaxAge: days * 24h}`.
  `Load` never resolves cwd; the caller hands it the anchor (Cwd Resolution Invariant).

`config_test.go` (untagged, `t.TempDir()` fixtures only, no git) covers:

- A valid `logger.yaml` returns its two values as bounds.
- An absent `_lyx/` and an absent `logger.yaml` under a present `_lyx/config/` both return bounds equal to `logger.DefaultRetentionBounds()`.
- The template's own resolved values equal `logger.DefaultRetentionBounds()`, pinning the compiled-in defaults to the template.
- Zero, negative, and non-integer values for each key each return an error; a missing key returns an error.

Register the new call site with the Config Strictness Invariant in the same commit:
add `"internal/loggerconfig"` to `configStrictnessDegradingSet` in `cmd/lyx/configstrictness_test.go`, and add `loggerconfig` to the degrading set in `CONSTRAINTS.md`'s Config Strictness Invariant.
In `docs/shared-libs/README.md`, add an `internal/loggerconfig` line after `internal/logger`'s: the `logger.yaml` loader for the trace-retention bounds, kept out of `internal/logger` to avoid the `configengine` import cycle.

**ImpactSummary:** Adds one degrading-set entry to the Config Strictness guard and its `CONSTRAINTS.md` list, plus one README line; no existing code path changes.

**Commit:** 2: loggerconfig: add logger.yaml with trace-retention bounds
