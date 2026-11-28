# Card 2 — beam-window-placement-setting

**Create:**
- `internal/beamsetting#WindowPlacementKey`
- `internal/beamsetting#WindowPlacementInherit`
- `internal/beamsetting#WindowPlacementOff`
- `internal/beamsetting#WindowPlacementSame`
- `internal/beamsetting#WindowPlacementNext`
- `internal/beamsetting#WindowPlacementValues`
- `internal/beamsetting#WindowPlacementModes`
- `internal/dobbyengine#WindowPlacementKey`
- `internal/dobbyengine#LookupWindowPlacement`

**Edit:**
- `internal/beamsetting/beamsetting.go#`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#CheckLocalValues`
- `internal/dobbyengine#checkTaskSetting`
- `internal/dobbyengine#Config`
- `internal/dobbyengine/tasksetting.go#`
- `internal/dobbyengine/template.yaml`
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine#lookupAsText`
- `internal/dobbyengine#TestLookupTaskSettings`
- `internal/dobbyengine#TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`
- `internal/dobbyengine#TestLoadConfig`
- `internal/configreg#TestPrepareRunSettings`
- `internal/boardcli#storeVerbs`
- `//pattern/PATTERN-config-layers.md`

**Uses:**
- `internal/boardcli/doc.go`

**Intent:** Add the per-task beam setting `beam.window_placement` (D7), read like `beam.run_watch`.

In `internal/beamsetting` declare the key `window_placement` and its values:
`""` (`WindowPlacementInherit`, hand over to the machine key), `off`, `same` and `next`.
`WindowPlacementValues` returns all four, for the per-task check.
`WindowPlacementModes` returns `off`, `same` and `next`, the values the machine key `desktop.window_placement` takes (card 3),
so both keys take their values from this one declaration.
The package comment's list of per-task settings names the new one.

In `internal/dobbyengine`:
- re-export the key as `WindowPlacementKey` beside `RunWatchKey`;
- classify it `configengine.CategoryNone` in `ConfigSpec`;
- add `window_placement: ""` to `template.yaml`, its comment saying:
  per task, which desktop lym places the task's terminal window on, `off`, `same` or `next`,
  and `""` hands over to `desktop.window_placement`; decided like beam.pull_request;
- add it to the key lists of `LoadConfig` and `CheckLocalValues`, so a resolved or run_settings value outside the four is refused on load (D7);
- give `checkTaskSetting` a `WindowPlacementKey` case taking `WindowPlacementValues`; `""` is a text value in that list, so the template's `""` passes;
- add `LookupWindowPlacement`, the only reader of the key, returning `lookupTaskSetting(chain, WindowPlacementKey)`;
- update the `Config` doc comment, the `tasksetting.go` file comment and `doc.go`'s list of per-task settings to name `beam.window_placement`.

Tests, all `tier0`, rows in existing tests:
- `lookupAsText`, the helper `TestLookupTaskSettings` and the refusal test dispatch through, gains a `WindowPlacementKey` case calling `LookupWindowPlacement`;
  without it the key would fall through to `LookupPullRequest`.
- `TestLookupTaskSettings`: a subtest like the run_watch one, with rows
  "the task's local file wins over its parent beam's worktree file" (`same` in the task's local layer over `next` in the beam's worktree file),
  "a task that sets nothing takes its beam's value over the prime's", "the hub default applies when no anchor sets the key",
  and "nothing set anywhere takes the template's `\"\"`".
  This pins D7's "a task `run_settings` value wins over its beam's, a beam's over the prime's".
- `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`: a row `window_placement left` in a worktree file,
  `wantValues: "[ off same next]"` as `fmt.Sprint` prints the four values.
- `TestLoadConfig` (dobbyengine): a row refusing a worktree `window_placement: sideways`.
- `TestPrepareRunSettings`: a row admitting `window_placement: next` into the beam local document,
  and a row refusing `window_placement: sideways` naming `window_placement`.
Together they add under 0.2 seconds to one `tier0` run.

`storeVerbs`' help text (`internal/boardcli/cli.go`) and `pattern/PATTERN-config-layers.md`'s "Per-task beam settings" rule
list `beam.window_placement` beside `beam.run_watch` (Review criterion).
`internal/boardcli`'s `TestHelpSchema_LeafCommands` still finds the strings it asserts, so it is unchanged.

**ImpactSummary:** One new per-task key on the existing lookup path; the beam module's load and run_settings check gain one key, and no consumer of the other keys changes.
