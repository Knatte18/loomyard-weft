# Card 3 — orch-config-modes

**Edit:**
- `internal/orchengine/config.go`
- `internal/orchengine/config_test.go`
- `internal/orchengine/template.yaml`
- `internal/orchengine/doc.go`
- `internal/orchcli/start_test.go`

**Create:**
- `internal/orchcli/config_refusal_integration_test.go`

**Uses:**
- `internal/orchcli/cli.go`
- `internal/orchcli/prime_integration_test.go`

**Intent:**
Add `cycle_mode` (D5) and narrow `permission_mode` to `bypass` (D10).

`Config` gains `CycleMode string` (`yaml:"cycle_mode"`) with exported constants for `clear` and `compact`, and a `Config.Mode()` accessor returning `compact` for an empty value.
`LoadConfig` keeps `configengine.LoadOrTemplate` (PATTERN-config-strictness) and, after the unmarshal, runs two value checks that return load errors:
`cycle_mode` other than empty, `clear` or `compact` is an error naming both accepted values;
`permission_mode` other than empty or `bypass`, `prompt` included, is an error naming the fix `set permission_mode: bypass or remove the key`.
An empty `permission_mode` resolves to `bypass` on the returned Config, so the spec `start` builds still carries `bypass`.
Update the `PermissionMode` field comment.

`template.yaml`: add `cycle_mode: compact` with a comment naming both values and what each does (compact keeps the session id and Remote Control link and writes no handoff file; clear runs the handoff, `/clear` and resume cycle); keep `permission_mode: bypass` with a comment saying it is the only accepted value and the orch never prompts per action.

`internal/orchengine/doc.go`: remove the operator-lever paragraph under "Permission mode and subagents" and state the bypass-only rule and the load error.

Because `internal/orchcli/cli.go`'s persistent pre-run loads the config before every verb, the refusal reaches `start`, `status`, `cycle`, `stop` and the watcher start before any pane is touched with no code change there.
The new integration test builds a hub fixture the way `prime_integration_test.go` does, writes an `orch.yaml` with `permission_mode: prompt`, and asserts `lyx orch status` and the watcher start each fail with the fix named; it is `integration`-tagged and its package's `TestMain` already isolates tmux and git.

Config tests: `cycle_mode` empty or absent is compact, `clear` and `compact` load, anything else errors naming both; `permission_mode` empty or `bypass` loads, `prompt` and another value error naming the fix.
`config_test.go`'s full-file case drops `permission_mode: prompt` for `bypass`, and `start_test.go`'s `TestStart_SpecShape` drops its `prompt` sub-case.

**ImpactSummary:** Adds two config value checks that make every `lyx orch` verb refuse a non-bypass permission_mode or an unknown cycle_mode; the template and the orchengine doc change with them.
