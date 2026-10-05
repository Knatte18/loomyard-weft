# Card 2 — shuttle-shell-wait-config

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#LoadConfig`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/config_test.go`

**Intent:**
D1's bound becomes a `shuttle.yaml` key: `background_shell_wait_min`, default 10.

`Config` gains `BackgroundShellWaitMin int` tagged `background_shell_wait_min`.
`template.yaml` carries `background_shell_wait_min: 10` with a comment in the file's own style: minutes a turn end waits on an outstanding background shell before it counts as a turn end, forks and declared shells excepted, and 0 or negative is a config error, not "unlimited".
`LoadConfig` returns an error naming the key and the value when it is not positive, so the value loads through shuttle's existing `LoadOrTemplate` path (PATTERN-config-strictness) and a missing key is filled from the template as every other key is.

Tests: the template default resolves to 10;
a config file setting 0, and one setting a negative value, each fail `LoadConfig` with an error naming `background_shell_wait_min`;
a positive override loads.

**ImpactSummary:** one new config key with a validated default; no existing key or reader changes.
