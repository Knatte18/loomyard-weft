```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. Map each behaviour the diff adds or changes to the test row that would fail if it broke.

**Behaviour → pinning test**

| Behaviour | Test |
|---|---|
| `Passages` and `FileEntries` slug and engine keying | `TestRenderer_EntryKeys` |
| `Show` text unchanged | `TestShowAndPassagesReadTheShippedLayerFirst`, `TestRender` |
| `cmd/lym` closure converting both into `Standing` | none (F1) |
| `StandingPassages` zero value cases | Claude `TestPrepare_AppendsStandingPassages` |
| Codex refusal names the passage slug | Codex `TestPrepare_AppendsStandingPassages` |
| Record write/read, unknown key, absent file, verdict, `MissingItems` | `TestManifest_Record` |
| Copy kept, absent record, blocked directory, unreadable record | `TestRunner_Start_KeepsTheManifestCopy` |
| No copy when the launch fails before its run record is saved | none (F10) |
| Sweep and strand lookup skip `manifests` | `TestSweepOrphans_AgeGuardAndLiveGuid`, `TestFindRunByStrand` |
| `ListedSkills`, `Plugin`, `Hook` | the two `TestUserSetup_*` tests |
| Claude manifest main items, guards, nudge, heddle entry, hand-written prompt | `claudeengine` `TestPrepare_WritesTheManifest` |
| Claude: no repository, nested anchor, user-setup error | none (F8) |
| Codex manifest main items, omitted hook, verdict | `codexengine` `TestPrepare_WritesTheManifest` |
| An item is dropped when its delivery text is not on the launch | only guards, nudge, skill load, unmarked file (F2) |
| Codex hand-written prompt; `Role`/`PromptEngine` on either engine | none (F7) |
| Manifest write failure refuses the launch (both engines) | none (F9) |
| `refuse_raw` guard item | none (F4) |
| Unrun hook warns nothing | `TestPrepare_UserSetup/UnrunHookWarnsOfNothing` |
| `NewWithSeams` hands seams to Codex | `TestNewWithSeams_CodexTakesTheGivenSeams` |
| `New` falls back to `RunPluginHook` | none at tier0; the plan states this bound (card 5), so no finding |
| `seatengine.Specs` | `TestSpecs_MatchTheStartedSpecs` |
| `Compare`, fork-deny gate, `UnrunHooks`, model-pin clearing, builder failure | `TestCompare` |
| `ConfiguredSandbox` | `TestSeats_CodexManifestHoldsClaudes` |
| `PromptItems` | `TestCompare/PromptItems` |
| `PrepareEach` appends the prompt items | none (F3) |
| `Unmet` for forks | `TestCompare/Unmet`, `TestReport` |
| `Unmet` for `spec.Requires` | none (F6) |
| Seat parity | `TestSeats_…`, except the unassisted burler reviewer (F5) |
| Producer parity and literal scan | `TestProducers_…`, `TestLaunchSpecLiterals_AreListed` |
| `report` results and temp-dir removal | `TestReport` |
| Verb envelope and exit code | `TestManifestCmd` |
| Verb's production wiring; stored verdict left out of output | none (F11) |

**Findings** (all class `design`)

- **F1, MEDIUM**, `cmd/lym/main.go:180-197`: no test runs the installed passage reader.
  - Dropping the `FileEntries` loop, or swapping it with `Passages`, passes every test, and real launches then list no `heddle-file/*` item.
  - Evidence: `cmd/lym/prerunseams_test.go:55` only checks that `InstallPassageReader` is called; the parity and engine tests install their own readers.

- **F2, MEDIUM**, `internal/shuttleengine/codexengine/manifest.go:118` and `claudeengine/manifest.go:86`: the read-back filter for text deliveries has no negative row.
  - This covers passages, instruction files, the pointer, the skill listing, plugin contexts and notices.
  - Replacing `inSetting(x)` or `appended(x)` with `x != ""` passes every row, because each candidate is always delivered there. A dropped delivery would then still be listed, and the parity test would not fail (the D1 acceptance criterion).
  - Evidence: `absent` lists in both engines' `manifest_test.go`.

- **F3, MEDIUM**, `internal/shuttleparity/parity.go:102`: nothing asserts that `PrepareEach` appends the `edit-directive` items.
  - Deleting the append drops the item from both engines, so the parity tests, `TestReport` and the verb tests all still pass.
  - Evidence: the `PrepareEach` rows in `parity_test.go` assert only errors and the prepared skills.

- **F4, MEDIUM**, `internal/shuttleparity/seats_test.go` (`newParityFixture`): the fixture writes no gate config, so the `refuse_raw` guard row is never compared.
  - Dropping Codex's raw-refuse hook passes the parity tests, against "dropping any one Codex delivery from `Prepare` fails the test".
  - Both engine manifest tests also call `shellguard.Rows(…, nil)`, so no test lists that item at all.
  - Evidence: the fixture's `files` map; `shellguard.Rows` appends the row only when `len(refuseRaw) > 0`; `claudeengine/manifest_test.go:30`, `codexengine/manifest_test.go:30`.

- **F5, MEDIUM**, `internal/shuttleparity/tables.go:383-389`: the unassisted burler reviewer is never in the seat catalog.
  - Both `SeatTables` calls set `opts.Assist`, and `withAssistant` replaces the chair's stencil with the assisted one.
  - So `burler-template-reviewer`, solo or fanned without an assistant, is never rendered or compared, although the criterion says every seat.
  - Evidence: `internal/burlerengine/assist.go:152-160`, `review.go:80`.

- **F6, LOW**, `internal/shuttleparity/parity.go:55`: no row gives `Unmet` a spec with `Requires`.
  - Starting `required` empty passes every test, and the verb would stop reporting the driver's or Merriam's unmet background wake.
  - Evidence: `TestCompare/Unmet` and `TestReport` use `ForkSubagents` only.

- **F7, LOW**, `codexengine/manifest.go:129`, `codexengine.go:274`, `claudeengine/manifest.go:175`: several record fields are unpinned.
  - No test covers Codex's hand-written prompt key.
  - No `Prepare` test asserts the record's `Role` or `PromptEngine`, which the verb prints.
  - Evidence: neither engine's `manifest_test.go` reads those fields, and the Codex table has no `HandWrittenPrompt` row.

- **F8, LOW**, `claudeengine/manifest.go:169,201`: three branches of the Claude manifest have no row.
  - An anchor with no repository (card 3 promises only the working directory's `CLAUDE.md` is listed and the launch goes on).
  - An anchor below the repository root, where the walk lists each `CLAUDE.md`.
  - A user-setup error, which is logged and must not block the launch.
  - Evidence: every row of `claudeengine/manifest_test.go` makes the anchor its own repository root, with a good setup.

- **F9, LOW**, `claudeengine/manifest.go:181`, `codexengine.go:300`: no test covers a failed manifest write refusing the launch, which cards 3 and 4 require.
  - Swallowing the error passes every test.

- **F10, LOW**, `internal/shuttleengine/run.go:741`: nothing pins that only a launch with a saved run record keeps a copy.
  - Moving `copyManifest` above `saveRunState` (line 728), or right after `Prepare`, passes `TestRunner_Start_KeepsTheManifestCopy`, whose rows all start successfully.

- **F11, MEDIUM**, `internal/shuttlecli/manifest.go:27-40,106`: the verb's production wiring is untested.
  - `TestManifestCmd` stubs `manifestReport`. Building the set with `shuttleengines.New` instead of `NewWithSeams` would run plugin hooks and the sandbox probe, against D10, and no test would fail.
  - The stubbed manifests carry no `Verdict`, and nothing asserts that the output omits it.
  - Evidence: `internal/shuttlecli/manifest_test.go`, `manifest.go:33`.

- **F12, LOW**, both engines' `manifest_test.go:28`: the expected guard keys are computed with `shellguard.Rows`, the helper `guardCandidates` itself uses (the rubric's tautological-test case).
  - A row lost from the table passes both tests. Literal row names such as the python and publish guards would be independent expected values.

Tool calls:
1. Guard defaults, the `shellguard.Rows` body, the remaining producer literals and `cmd/lym` reader tests. The context lacked the `deny_python` default, the raw-row condition, those literals and any existing closure test.
2. Line numbers for the findings. The context held the code but not its line positions.
