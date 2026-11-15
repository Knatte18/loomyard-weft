```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour, the test that pins it**

1. `runCommand(cmd)` reads `RunProcessAnnotation`: pinned by `cmd/lym.TestRunCommand_MarksOnlyARunsOwnVerbs` (`cmd/lym/main_test.go`). Its rows are `board get`, `board list`, `machine name`, `selfreport create`, `adopt`, `beam start` and `reed attach` false; `batten run`, `batten step`, `loom run`, `shed run` and `beam dobby` true. The `shed run`, `shed step` and `beam dobby` annotations are covered through those rows. Breaking any of the three annotation sites fails a row, but there is no `loom step` row.
2. `InstallNetworkGate` sets `LYM_RUN_PROCESS` when `runCommand` is true: pinned by the mark assertion in `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` (`cli_test.go:~484`), row `RunCommandHeldAndMarksItsChildren`. The same assertion checks that the mark is not set when `runCommand` is false.
3. `heldByPause` classification, each branch pinned by a `childTests` row in `internal/tendercli/cli_test.go`:
   - Operator verb passes: `OperatorVerbPassesHubEntry`, `OperatorVerbPassesItsPairsEntry`.
   - Orch strand passes: `OrchPassesHubEntry`.
   - Non-orch strand agent held: `AgentHeldByItsPairsEntry`.
   - Unparsable strand name held: `UnparsableStrandHeld`.
   - Inherited mark held: `ChildOfARunCommandHeld`.
   - Run command held: `RunCommandHeldAndMarksItsChildren`.
   - Detached child keeps its hold: the four existing `DeferrableUnderHubEntry` and `HubScope*` rows.
   - `ParentLocked` passing an owner entry: `ParentLockedPassesOwnerEntry`.
4. `Gate.ExemptAll` lets every live entry pass in the waiter: pinned by the existing `internal/pauserecord/hold_test.go` ExemptAll tests (line 437 region). The diff adds no new wait behaviour.
5. `stripIdentityEnv` drops `LYM_RUN_PROCESS`: pinned by `TestStripIdentityEnv_DropsNameParentAndRunMarkOnly` (`lifecycle_test.go`). It also pins that the longer `LYM_PARENTAL` key is kept. Its `//testtiming:keep` reason matches the report's "not judged, kept" note.
6. `tmuxkit.ServerEnviron` drops `LYM_RUN_PROCESS`: pinned by `TestEnvScrub_MatchesTmuxkitServerEnviron` (`shuttleengines_test.go`). The row `LYM_RUN_PROCESS=1` was added to both the input and the expected filter. The kit's literal `"LYM_RUN_PROCESS"` is not tied to `pauserecord.RunProcessEnv` by any test, because the kit cannot import `pauserecord`. The test compares the kit against a hard-coded literal, so it catches a drifted copy only if the literal there is right. The reed copy uses the constant, so one string would have to drift in two places at once for this to go unnoticed.
7. A paused hub does not hold `board get`, `board list`, `machine name`, `selfreport create` and `lym s`: pinned only by composition. `TestRunCommand_MarksOnlyARunsOwnVerbs` shows those verbs read false. The `OperatorVerbPassesHubEntry` and `OperatorVerbPassesItsPairsEntry` rows show an unmarked process passes. No test installs the gate through the real root's `PersistentPreRun` against a paused hub and runs one of those verbs.
8. A verb that waits prints its held line at once: the diff changes nothing here and adds no test. The existing waiter is unchanged.
9. The `lym s` second hold in `internal/battenshed/innerrun_pause.go`: the diff changes nothing and adds no test. Behaviour: none, no test.
10. The mark set before the hub check in `InstallNetworkGate`: no row combines `runCommand=true` with a call outside a hub. The outside-hub loop in the test passes `false`.

**Findings**

- F-T1 · LOW · design · `internal/tendercli/cli_test.go:~362-420`: no row covers `runCommand=true` outside a hub, where `os.Setenv` runs before the hub check and no gate is installed. The mark set with no gate, and its leak into later tests of the same process, is untested. Evidence: `gate.go:55-58` sets the mark before the hub resolution, and the outside-hub loop passes `false`.
- F-T2 · MEDIUM · design · `cmd/lym/main_test.go:196-229` and `internal/tendercli/cli_test.go:~440-490`: the board bullet asks for a test that a paused hub does not hold the five verbs. The diff proves it only by composition, as in item 7. No test drives a real verb's pre-run through `newRoot()` and `InstallNetworkGate` and checks that `GateCovered()` is false under a live hub entry. If `PersistentPreRun` stopped passing `runCommand(cmd)`, or passed it wrongly, every test would still pass. Evidence: the `runCommand(cmd)` call at `cmd/lym/main.go:157` is covered by no test.
- F-T3 · MEDIUM · scope · `internal/battenshed/innerrun_pause.go:56-82` (unchanged by the diff): behaviour 9 above. The board entry names `lym s` as held at this second place, and nothing in the diff or its tests addresses it. If `lym s` still reaches it, the unmet case is also untested.
- F-T4 · NIT · consistency · `internal/testkit/tmuxkit/tmuxkit.go:540` and `internal/shuttleengines/shuttleengines_test.go:~230`: `"LYM_RUN_PROCESS"` is a literal in both. No test ties either to `pauserecord.RunProcessEnv`. See item 6.

Test economy: `TestRunCommand_MarksOnlyARunsOwnVerbs` is "unique" in the report. The `shed run` and `loom run` rows reach the same annotation site as `batten run`, but they pin the three entry points separately, so I raise no finding.

Tool calls: none; the manifest already held the diff and the tests.
