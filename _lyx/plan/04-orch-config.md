# Card 4 — orch-config

**Create:**
- `internal/orchengine/config.go`
- `internal/orchengine/template.go`
- `internal/orchengine/template.yaml`
- `internal/orchengine/config_test.go`
- `internal/orchengine/seam_enforcement_test.go`

**Edit:**
- `internal/configreg/configreg.go`
- `internal/configreg/configreg_test.go`
- `cmd/lyx/configstrictness_test.go`

**Uses:**
- `internal/shuttleengine/config.go`
- `internal/shuttleengine/template.go`
- `internal/battenshed/seam_enforcement_test.go`

**Intent:**
Start `internal/orchengine` with its config (D15), registered like every other module config and loaded degrading.

`config.go` declares `Config` with yaml keys `model`, `effort`, `threshold_tokens`, `idle_grace_s`, `handoff_timeout_s` and `poll_interval_ms`, and `LoadConfig(baseDir, module string) (Config, error)` over `configengine.LoadOrTemplate(baseDir, module, []byte(ConfigTemplate()))`, mirroring `shuttleengine.LoadConfig`.
Three accessors return `time.Duration` values — `IdleGrace()`, `HandoffTimeout()` and `PollInterval()` — and `Threshold()` returns the token count.
Each floors a non-positive value back to the template default, the way shuttle floors its poll interval, so a zero typo can never make the watcher cycle on every turn or busy-spin.

`template.yaml` is the embedded template, one commented line per key:
`model` and `effort` empty (provider default; effort values are shuttle's engine vocabulary), `threshold_tokens: 150000` (D8), `idle_grace_s: 30` (D9), `handoff_timeout_s: 600` (D11) and `poll_interval_ms: 2000`.
The `threshold_tokens` comment says it is an absolute count because the transcript does not state the model's window, and that a value above the model's auto-compaction point lets Claude Code compact first.
`template.go` embeds it behind `ConfigTemplate()`.

`seam_enforcement_test.go` mirrors `internal/battenshed`'s allowlist scan over this package's production files.
The allowlist is `configengine`, `shuttleengine`, `state`, `lock`, `logger`, `stencil`, `stencilstore` and `gopkg.in/yaml.v3`.
It separately names two denied imports so a violation reports the rule it breaks: `internal/lyxcwd` (Told-Geometry Invariant) and `internal/shuttleengine/claudeengine` (Shuttle Provider-Seam Invariant).

`config_test.go` covers the template resolving when no config file exists, a present file overriding values, an invalid file erroring, and each accessor flooring a zero or negative value.

Register the module: add `{Name: "orch", Template: orchengine.ConfigTemplate}` to `configreg.Modules()` in alphabetical position (between `models` and `reed`), extend `TestNames`' pinned list to match, and add `internal/orchengine` to `configStrictnessDegradingSet` in `cmd/lyx/configstrictness_test.go`.
`CONSTRAINTS.md`'s degrading list is updated in card 13.

**ImpactSummary:** One new registry entry reaches `lyx config`'s listing, reconcile and menu; the two edited tests only extend pinned sets.
