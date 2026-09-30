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
- `plan:internal/loggerconfig#ConfigPath` -> `func ConfigPath(anchorPath string) string`

**Uses:**
- `plan:internal/logger#RetentionBounds`
- `plan:internal/logger#DefaultRetentionBounds`
- `internal/configengine#LoadOrTemplate`
- `internal/configengine#ConfigFile`
- `internal/batcher#Active`

**Intent:**
Add `internal/loggerconfig`, the module owning `_lyx/config/logger.yaml` (decision record D4, D5).
It lives outside `internal/logger` because `internal/configengine` imports `internal/logger`, so the logger cannot load its own config without a cycle.
Follow `internal/batcher`'s config shape (`template.yaml` embedded by `template.go`, a `config.go` calling `configengine.LoadOrTemplate`).

- `template.yaml` carries `trace_retention_count: 200` and `trace_retention_days: 14`, each with a trailing comment stating what it bounds, in the style of the other module templates.
- `template.go` embeds it and exposes `ConfigTemplate()`.
- `config.go` declares the package doc, a `moduleName = "logger"` constant, and `Load(anchorPath)`:
  it calls `configengine.LoadOrTemplate(anchorPath, moduleName, []byte(ConfigTemplate()))`, unmarshals both keys into `yaml.Node` fields, and returns an error naming the key when either node is not a scalar whose `ShortTag()` is `!!int`, or when its decoded `int` is zero or negative.
  The tag check is what rejects a float: yaml.v3 decodes a `!!float` such as `1.5` or `14.0` straight into an `int` field by truncation, with no error, so a plain `int` field would accept it.
  On success it returns `logger.RetentionBounds{Count: count, MaxAge: days * 24h}`.
  `Load` never resolves cwd; the caller hands it the anchor (Cwd Resolution Invariant).
- `config.go` also exposes `ConfigPath(anchorPath)`, returning `configengine.ConfigFile(anchorPath, moduleName)`, so the CLI's exit hook can name the file in its Warn record without knowing the module name.

`config_test.go` (untagged, `t.TempDir()` fixtures only, no git) covers:

- A valid `logger.yaml` carrying both keys at non-default values returns those two values as bounds.
- An absent `_lyx/` and an absent `logger.yaml` under a present `_lyx/config/` both return bounds equal to `logger.DefaultRetentionBounds()`.
- The template's own resolved values equal `logger.DefaultRetentionBounds()`, pinning the compiled-in defaults to the template.
- For each key, a zero, a negative, a float (`1.5` and `14.0`), and a string value each return an error naming that key.
  Each fixture is a valid `logger.yaml` carrying both keys with exactly the key under test changed, so `configengine`'s missing-key check cannot satisfy the case in its place.
- A missing key returns an error.
- `ConfigPath(anchor)` equals `<anchor>/_lyx/config/logger.yaml`.

Register the new call site with the Config Strictness Invariant in the same commit:
add `"internal/loggerconfig"` to `configStrictnessDegradingSet` in `cmd/lyx/configstrictness_test.go`, and add `loggerconfig` to the degrading set in `CONSTRAINTS.md`'s Config Strictness Invariant.
In `docs/shared-libs/README.md`, add an `internal/loggerconfig` line after `internal/logger`'s: the `logger.yaml` loader for the trace-retention bounds, kept out of `internal/logger` to avoid the `configengine` import cycle.

**ImpactSummary:** Adds one degrading-set entry to the Config Strictness guard and its `CONSTRAINTS.md` list, plus one README line; no existing code path changes.

**Commit:** 2: loggerconfig: add logger.yaml with trace-retention bounds
