# Card 1 — run-watch-setting

**Create:**
- `plan:internal/beamsetting#RunWatchKey` -> `const RunWatchKey = "run_watch"`
- `plan:internal/beamsetting#RunWatchStops` -> `const RunWatchStops = "stops"`
- `plan:internal/beamsetting#RunWatchRows` -> `const RunWatchRows = "rows"`
- `plan:internal/beamsetting#RunWatchOff` -> `const RunWatchOff = "off"`
- `plan:internal/beamsetting#RunWatchValues` -> `func RunWatchValues() []string`
- `plan:internal/dobbyengine#RunWatchKey` -> `const RunWatchKey = beamsetting.RunWatchKey`
- `plan:internal/dobbyengine#RunWatchStops` -> `const RunWatchStops = beamsetting.RunWatchStops`
- `plan:internal/dobbyengine#RunWatchRows` -> `const RunWatchRows = beamsetting.RunWatchRows`
- `plan:internal/dobbyengine#RunWatchOff` -> `const RunWatchOff = beamsetting.RunWatchOff`
- `plan:internal/dobbyengine#LookupRunWatch` -> `func LookupRunWatch(chain TaskSettingChain) (string, error)`
- `plan:internal/dobbyengine#TestCheckLocalValues` -> `func TestCheckLocalValues(t *testing.T)`

**Edit:**
- `internal/beamsetting/beamsetting.go`
- `internal/dobbyengine#Config`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#CheckLocalValues`
- `internal/dobbyengine#checkTaskSetting`
- `internal/dobbyengine/tasksetting.go`
- `internal/dobbyengine/template.yaml`
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine#defaultsWith`
- `internal/dobbyengine#TestLoadConfig`
- `internal/dobbyengine#TestLookupTaskSettings`
- `internal/dobbyengine#TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`
- `pattern/PATTERN-config-layers.md`

**Uses:**
- `internal/dobbyengine#writeBeamLayer`
- `internal/dobbyengine#keyName`

**Intent:**
Add the per-task key `beam.run_watch` (D2) and the two hub keys `beam.gate_hold_factor` and `beam.gate_hold_min` (D6), so later cards read them.

`beamsetting` declares `RunWatchKey`, its three values and `RunWatchValues()` (in the order stops, rows, off), and its package doc names `run_watch` among the per-task settings.
`dobbyengine` aliases them in `config.go` beside the `land_notice` aliases.
`ConfigSpec` gives `run_watch`, `gate_hold_factor` and `gate_hold_min` `configengine.CategoryNone`.
`checkTaskSetting` picks `beamsetting.RunWatchValues()` as the allowed list for `RunWatchKey`, so its existing refusal names the full key `beam.run_watch` (through `keyName`), the value, the three values and `lym config beam --set run_watch=<value>`.
`LoadConfig` and `CheckLocalValues` add `RunWatchKey` to the per-task keys they check, so a bad value is refused where a bad `land_notice` is; their doc comments name the key.
`LookupRunWatch` returns `lookupTaskSetting(chain, RunWatchKey)`, the text value; its doc says it is the only reader of the key.
`tasksetting.go`'s file comment and `Config`'s doc comment name `run_watch` among the per-task keys, and so does the file comment of `tasksetting_test.go`, which holds the lookup tests below.

`Config` gains `GateHoldFactor float64` (`yaml:"gate_hold_factor"`) and `GateHoldMin int` (`yaml:"gate_hold_min"`).
`LoadConfig` refuses a negative `gate_hold_factor` and a `gate_hold_min` below 1, each marked `configengine.MarkInvalid`, naming the full key, the value, the range ("must not be negative", "must be at least 1") and the way forward `lym config beam --set <key>=<value>`, as the negative-int refusal does.
The input that reaches each refusal is a worktree layer setting `gate_hold_factor: -0.5` or `gate_hold_min: 0`; no upstream check rejects either, since the template validation checks only key presence.

`template.yaml` gains, each with a trailing comment:
`run_watch: "stops"` (per task: which changes of a task's run dobby mails the orch as a `[run watch]` line: stops, rows or off; decided like beam.pull_request),
`gate_hold_factor: 1.5` (dobby notices a gate slot a run holds past this many times its tier's timeout_min, in awake time; 0 sends no gate-hold notice) and
`gate_hold_min: 90` (the minutes the gate-hold bound is for a holder with no tier or a tier with no timeout; at least 1).
`doc.go`'s landing-notice bullet that lists the per-task settings and their lookups names `run_watch` and `LookupRunWatch`.
`pattern/PATTERN-config-layers.md`'s per-task rule line names `beam.run_watch` beside the three keys it lists.

Tests:
- `defaultsWith` carries the template's `GateHoldFactor: 1.5, GateHoldMin: 90`.
- `TestLoadConfig` gains rows: a worktree layer's `run_watch: rows` loads; a local layer's `run_watch: "off"` loads; a `run_watch: all` is refused naming `run_watch`, `all`, the values `[stops rows off]` and `lym config beam --set run_watch=`; a `gate_hold_factor: 0` resolves as set; a `gate_hold_min: 30` resolves as set; a `gate_hold_factor: -0.5` is refused naming `gate_hold_factor`, `must not be negative` and `lym config beam --set`; a `gate_hold_min: 0` is refused naming `gate_hold_min`, `at least 1` and `lym config beam --set`.
- `TestLookupTaskSettings` gains a subtest for `run_watch` over the same chain shapes, since its on/off rows do not fit the key's values: the task's local `rows` over its worktree `off`; a task that sets nothing taking the parent's `off` over the prime's `rows`; the hub default `rows` when no anchor sets the key; and nothing set anywhere taking the template's `stops`.
- `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake` gains the row `run_watch all in a worktree file`, whose error names the path, `run_watch`, `all`, the values `[stops rows off]` and `lym config beam --set run_watch=`.
- `TestCheckLocalValues`, a new table test, since no test calls `CheckLocalValues`, the check `internal/configreg/configreg.go` runs over a task's `run_settings`: a document with `run_watch: rows` passes; one with `run_watch: all` is refused, marked `configengine.ErrInvalid`, naming `run_watch`, `all`, `[stops rows off]` and `the task's run_settings`; one with `land_notice: all` is refused the same way, so the table pins the existing keys beside the new one.

**ImpactSummary:** Adds keys only; the per-task checks gain one key and `LoadConfig` two range refusals, and `Config`'s only struct-literal readers are `defaultsWith` and the dobby's loads, so no caller changes.

**Commit:** 1: Add beam.run_watch and the gate-hold keys to beam.yaml
