# Card 3 — redraw-settle-config

**Edit:**
- `internal/shuttleengine/config.go`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/config_test.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/startup_test.go`

**Intent:**
D11: the post-Enter redraw wait moves into `shuttle.yaml`.

`shuttleengine.Config` gains `SubmitRedrawSettleMS int` under the yaml key `submit_redraw_settle_ms`, and `template.yaml` gains `submit_redraw_settle_ms: 300` directly below `submit_settle_ms`, with an inline comment in the style of its neighbours: the wait after an Enter before the input box is read to confirm the send, 0 reads at once, negative is a config error.
`LoadConfig` refuses a negative value with an error naming the key and the value, as it does for `submit_settle_ms`.
The template default keeps `LoadOrTemplate` callers degrading cleanly (`PATTERN-config-strictness`).

`claudeengine.Claude` gains a redraw-settle field: `New` sets the 300 ms default (the current `submitRedrawSettle` constant becomes that default), `NewFromConfig` reads `cfg.SubmitRedrawSettleMS`, and `SubmitSettle` returns the field.

Tests:
the existing negative-value table in `config_test.go` gains rows for `submit_redraw_settle_ms`, and a template load asserts the 300 default;
the settle table in `startup_test.go` (the one pinning `New()` against `NewFromConfig`) gains `SubmitSettle` assertions for the default and a configured value, including 0.

**ImpactSummary:** Adds one config key read by claudeengine's InputBoxReader capability; default behavior unchanged.
