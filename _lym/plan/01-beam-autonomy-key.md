# Card 1 — beam-autonomy-key

**Create:**
- `plan:internal/beamsetting#AutonomyKey` -> `const AutonomyKey = "autonomy"`
- `plan:internal/dobbyengine#AutonomyKey` -> `const AutonomyKey = beamsetting.AutonomyKey`
- `plan:internal/dobbyengine#LookupAutonomy` -> `func LookupAutonomy(chain TaskSettingChain) (bool, error)`

**Edit:**
- `internal/dobbyengine#Config`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#CheckLocalValues`
- `internal/dobbyengine#checkTaskSetting`
- `internal/dobbyengine/template.yaml`
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine/tasksetting.go`
- `internal/dobbyengine#TestLookupTaskSettings`
- `internal/dobbyengine#TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`
- `internal/configreg#TestPrepareRunSettings`
- `internal/boardcli#storeVerbs`
- `pattern/PATTERN-config-layers.md`

**Uses:**
- `internal/dobbyengine#lookupTaskSetting`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/boardcli/doc.go`

**Intent:** D1 and D11: `autonomy` becomes beam.yaml's third per-task setting, read only through dobbyengine's one lookup.

- `internal/beamsetting` declares `AutonomyKey`, and dobbyengine re-exports it beside `PullRequestKey` and `LandNoticeKey`.
- `template.yaml` gains `autonomy: false` with a comment in the style of its siblings:
  per task, whether the orch settles every halt of the task's runs and reaches the operator only for a severe halt or a merge-ready beam;
  decided like `pull_request`.
- `ConfigSpec` classifies the key `configengine.CategoryNone`, so the local layer a task's `run_settings` land in may hold it
  (PATTERN-config-layers; `TestModules_EveryTemplateKeyClassified` fails without it).
- `checkTaskSetting` gains the autonomy branch: the value must be a YAML boolean, and it returns `"true"` or `"false"`.
  Any other value is refused, marked `configengine.ErrInvalid`, naming the key, `where`, the value, "must be true or false" and the `lym config beam --set autonomy=<true|false>` way forward.
- `LoadConfig` and `CheckLocalValues` add `AutonomyKey` to the keys they run through `checkTaskSetting`,
  so a resolved stack or a task's run_settings holding `autonomy: "yes"` is refused when written.
- `LookupAutonomy` returns `lookupTaskSetting(chain, AutonomyKey) == "true"`.
  An envsource marker in a winning value expands as for the other keys.
- `Config`'s comment and `doc.go`'s per-task bullet name `autonomy` beside `pull_request` and `land_notice` and `LookupAutonomy` beside the other lookups.
  The file comments atop `tasksetting.go` and `tasksetting_test.go`, which name the two old keys as the per-task settings, name `autonomy` too.
- `storeVerbs`'s `run_settings` help line names `beam.autonomy` beside `beam.pull_request` and `beam.land_notice`.
- `pattern/PATTERN-config-layers.md`'s "Per-task beam settings" rule line names `beam.autonomy` beside the two keys it lists.

Tests, all tier0 in existing functions:

- `TestLookupTaskSettings` runs its table under `AutonomyKey` too, with `on` read as `true` and `off` as `false`:
  `settingDocument` writes the autonomy value unquoted, and `lookupAsText` maps `LookupAutonomy`'s answer back to on or off.
  Its existing rows then pin D1's "a worktree layer of false over a default layer of true resolves false"
  (`the prime's own file applies when nothing nearer sets the key`, prime `off` under hub `on`)
  and D11's task-over-parent rows in both directions, the parent fallback, the hub default and the template's `false`.
  The rows are the shared table, so the third key adds no row: each row under `autonomy` pins the boolean mapping at that place of the lookup.
  Its "nothing set anywhere takes the template's values" subtest asserts `LookupAutonomy` answers false.
- `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake` gains rows `autonomy` `"yes"` in a worktree file and `autonomy` `"on"` in a local file,
  asserting the path, key, value and `lym config beam --set autonomy=` way forward.
- `TestPrepareRunSettings` gains an accepted row `{"beam":{"autonomy":true}}`, whose wanted local document is `autonomy: true`,
  and a refused row `{"beam":{"autonomy":"yes"}}` with `wantModule` `beam`, pinning D1's run_settings criterion.

**ImpactSummary:** Adds one per-task key to beam.yaml's template, spec, checks and lookup; the two existing per-task keys and their callers are unchanged, and the only new caller is card 3's reader.
