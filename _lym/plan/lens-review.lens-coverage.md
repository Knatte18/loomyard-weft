```yaml
lens: coverage
outcome: reported
```

Coverage lens: whether each decision, acceptance criterion and new branch has a card and a test row that would fail if it broke.

1. **BLOCKING — D7, Card 8.** `lym reed button` still runs reed's root pre-run: it resolves the location, loads the config and builds the geometry.
   The skip list in `internal/reedcli#Command`'s `PersistentPreRunE` names `reed`, `watchdog`, `switch`, `follow` and `window`. Card 8 adds `button` only to `AddCommand`, not to that skip list.
   D7 says the verb "reads no config and no reed state". As planned, a `reed.yaml` that fails to load makes every click abort silently: the verb's output is discarded and its exit forced to zero.
   No row reaches it: the `TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository` rows only refuse missing flags, and `TestRunButton_RealTmux` calls `runButtonVia` directly, bypassing cobra.
   Evidence: `internal/reedcli/cli.go:141-143`; Card 8 Intent ("`Command` registers it after `followCmd`").
2. **MEDIUM — D7 acceptance criterion "with the strand-name variables unset".** The plan relies on `stripAgentNameEnv` and has no row that observes the clicked command's environment.
   `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` records only `pwd` and the arguments. `runButtonScript` inherits the verb's environment, and nothing checks that `LYM_STRAND_NAME` or `LYM_PARENT` is absent.
   Evidence: Card 8 Intent (`runButtonScript` "sets no caller identity"); Card 9 test bullets.
3. **MEDIUM — D7 failed-spawn branch.** Step 7 of `runButtonVia` names "a spawn failure or a non-zero exit". Only the exit-3 row is planned; no row has the stub `run` return an error.
   Evidence: Card 8 step 7 and its `TestRunButtonVia` rows.
4. **MEDIUM — D8 acceptance criterion "both templates hold the task, beam and prime groups as listed".**
   - `TestLoadConfig_TemplateDefaultsResolve` asserts only the `task` labels, the `agent` run and an empty `prime`. The `beam` group (`code`, `term`, `agent`, `distill`, `refresh`), the three `confirm` texts and CANCEL's `run` go unasserted.
   - It reads the GOOS-selected `ConfigTemplate()`, so `template_windows.yaml` is never checked on a Linux gate.
   - Possible fix: read both YAML files from the package directory in a tier0 test.
   Evidence: Card 2 test bullets; `internal/reedengine/template.go` (GOOS-embedded).
5. **MEDIUM — D4 `{worktree}` is the anchor, not the worktree root.** The `Worktree` value is asserted only on the prime row of `TestReedGeometry`, whose `anchorRel` is `"."`, where anchor and worktree root coincide.
   The subpath-anchored task row exists, but the card does not call `Buttons` on it, so swapping in `l.WorktreePath()` passes.
   Evidence: `internal/hubgeom#TestReedGeometry` rows (`sub/dir` task, `.` prime); Card 4 test bullets.
6. **MEDIUM — Card 7's empty-name branch.** "An empty name unsets the option" is a new branch with no row. `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` gains the set call only. The Warn-on-failure path is unreached too.
   Evidence: Card 7 Intent and test bullets.
7. **MEDIUM — Card 4's beam-resolver error returns.** "A failed prime-name, parent or anchor read is returned wrapped" has no row: the tier1 `TestGeometryOverRealHub` steps are happy paths only.
   The task-with-no-origin-record case reading `ButtonPlaceTask` is not asserted either; only the `beam:true`, plain-record and unparseable-record rows are.
   Evidence: Card 4 Intent and test bullets.
8. **MEDIUM — Card 8 branches with no row:**
   - an unresolvable executable on the confirm path (Warn, runs nothing);
   - a spec value that is not valid JSON (only "unreadable" and "empty" rows are planned);
   - a range whose suffix after `lymbtn` is not a number;
   - a `@lym_strand_name` read that fails (only the empty answer is planned).

   Evidence: Card 8 steps 1, 2, 3 and 5 against its `TestRunButtonVia` row list.
9. **LOW — D1 acceptance criterion "after an older binary re-pins `status-format[1]`, the newest binary's next reed op restores it".** The planned `TestBindingsBuildStamp` row counts only the `status-format[2]` pins, so dropping `[1]` (or `[0]` and `status-style`) from `statusFormatPinArgvs` still passes.
   The repin's failure Warn is also unreached.
   Evidence: Card 5 test bullet `OpLockRepinsStaleBindings`.
10. **LOW — D2 invalid-entry rules.** `TestPlaceButtons` covers `#` in label, `%` in confirm and a tab in label, but not `#` or `%` in the other field, nor `{strand}` in `confirm` without `window: agents`. An implementation checking one field per character passes.
    Also unspecified: whether a malformed `prime` group warns before falling back to `beam`, and an absent group producing no Warn. Neither has a row.
    Evidence: Card 3 `buttonEntryProblem` list against its test list.
11. **LOW — D2 acceptance criterion "`lym reed up` succeeds" with a bad button.** No row boots a session through `Up` with a mapping group in `reed.yaml`. The plan argues it holds because `pinButtonRowLocked` is non-fatal.
    Evidence: Card 6 test list (`pinButtonRowLocked` driven directly).

Tool calls:
- `internal/reedcli/cli.go:118-180`: the context held only the `AddCommand` line, not the pre-run skip list.
- `fabricengine.ConfigDirs` and `configengine/config.go`'s FillMissing wrapping: to confirm the Board+Anchor dirs and that `buttons: null` reaches ErrInvalid. Both hold.
- `internal/logger` durable-level docs: to confirm Info reaches the run directory's log at default verbosity. It does.
