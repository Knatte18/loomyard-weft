# Card 14 — loom-idle-timeout-key

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`

**Intent:** `loom.yaml` gains `step_idle_timeout_min`, the watchdog window of D4, with a default of 60 and a floor of one minute.
`Config` gains `StepIdleTimeoutMin int` under that tag; `template.yaml` carries `step_idle_timeout_min: 60` with a comment naming it as the minutes a `--until-stop` step may show no activity before the loop kills it.
`LoadConfig` floors a value below 1 to 1 with one Warn naming the key and the floor, never refusing, as `PATTERN-daemon-wakeups` requires of a configured interval.
The hub reconcile seeds the key into every worktree's `loom.yaml` on the next binary change through `hubreconcile.Ensure`, which already adds a template key a file lacks, so no seeding code is written.
`config_test.go`'s load table gains rows for the default, an explicit value and a floored zero; the template-tag test covers the new tag by construction.

**ImpactSummary:** One additive config key; every existing `loom.yaml` loads as before until the reconcile seeds the key, and no reader consumes it until card 15.
