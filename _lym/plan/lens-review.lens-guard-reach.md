```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether the guards, refusals, branches and test rows the cards add can be reached.

1. **BLOCKING**, Card 2 (D2/D3, the desktop machine-layer row with a non-empty mapping): the row `buttons: {task: {label: x}}` cannot reach the "different shape" refusal it asserts.
   - `yamlengine.LeafPaths` steps into a non-empty mapping, so the unit is `buttons.task.label`, not `buttons.task`.
   - `nodeAt` returns nil when a step is not a mapping (`internal/configengine/layers.go:391-410`), and the template's `task` is a list. So `templateValueAt` returns nil.
   - `ValidateMachine` therefore takes the earlier branch at `internal/configengine/layers.go:341-343`: `key "buttons.task.label" is not in the desktop template`. The shape branch at `layers.go:345-347` is never reached.
   - The load still fails with `ErrInvalid`, which D2 requires, but the row's assertion on `buttons.task` and "different shape" fails.
   - Only the empty-mapping row `buttons: {task: {}}` reaches the shape branch, because `LeafPaths` counts an empty mapping as one unit.
   - Fix: assert that the error names `buttons.task` (both messages contain it) and `ErrInvalid`, and expect "different shape" only on the empty-mapping row.

2. **MEDIUM**, Card 2 (reed local-layer refusal row): if the row copies `LocalExecutableKeyIsRefused` literally, it checks for the quoted string `"buttons"`, and that check fails.
   - For `buttons: {task: []}`, `ValidateLocal` refuses the unit `buttons.task`. The message is `key "buttons.task" is executable…` (`internal/configengine/layers.go:298-300`).
   - That message never contains `"buttons"` with a closing quote.
   - Fix: assert on `buttons.task`, or on `"buttons` without the closing quote.

3. **LOW**, Card 8 (D7 failure message): only `#` is escaped in the status message, so a `%` in it still gets expanded.
   - tmux's `display-message` runs the message through strftime as well as format expansion. A `%` in the first output line ("100% done") or in a directory name is expanded as a time field, the same problem D2 bars `%` in labels for.
   - The card doubles only `#`. Double `%` as well, and add a `%` value to the `TestRunButtonVia` row.

4. **MEDIUM**, Card 8 (D6/D7 `{strand}` read at click time): the real lookup of the clicking client's active pane is never run against tmux.
   - The guard "an empty `@lym_strand_name` shows the agent-pane message" depends on `display-message -p -c &lt;client&gt; '#{@lym_strand_name}'` resolving the clicking client's active pane. The verb runs it as a separate tmux client process launched by `run-shell -b`.
   - Only the tier0 fake answers that read.
   - Neither `TestRunButton_RealTmux` (no `{strand}` entry) nor `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` (clicks only `HALT`) runs it against real tmux. So neither the guard nor the filled strand value is shown to be reachable on a real server.
   - Fix: add a `{strand}` entry to the tier2 test, clicked once from a strand's window and once from the shell window.

The other guards trace to reachable states:
- **Card 3:** every skip in `placeButtons`.
- **Card 4:** the origin-read error and the absent-beam anchor.
- **Card 6:** the nil closure, closure error, unresolvable shell, `Beam` error and empty `Anchor`.
- **Card 8:** a changed spec, an unreadable or empty spec, a bad carried strand, a hash mismatch and a gone directory. tmux's `run-shell -c` falls back to the home directory, so the verb still starts and reaches its own directory check.
- **Card 9:** `buttonClick` false.

Tool calls:
- Read `internal/configengine/config.go`'s template fill and `fabricengine.ConfigDirs`: I needed to confirm that `buttons: null` reaches a shape mismatch marked `ErrInvalid`, and which config dirs card 4's closure loads.
- Read `nodeAt` and `shapeMismatch`: I needed which `ValidateMachine` branch a non-empty group mapping reaches.
