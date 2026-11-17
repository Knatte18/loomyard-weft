```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per-behaviour list**

- Heddle passage and file-entry keys (slug, engine key; `Engine` on single-engine file targets): `heddleengine.TestRenderer_EntryKeys`, with the `TestRender` and `TestShowAndPassagesReadTheShippedLayerFirst` updates. Pinned.
- Manifest record (write, verdict, unknown key still reads, absent file, `MissingItems`): `shuttleengine.TestManifest_Record`. Pinned.
- Copy kept after a done run's cleanup, blocked or undecodable record never fails the run: `TestRunner_Start_KeepsTheManifestCopy`. Pinned.
- The orphan sweep and `FindRunByStrand` skip `manifests`: `TestSweepOrphans_AgeGuardAndLiveGuid` and the new `TestFindRunByStrand` row. Pinned.
- Claude manifest read back from the artifacts: `claudeengine.TestPrepare_WritesTheManifest`. It pins prompt-engine, guards, notices, nudge, skill load, native files, listing, native plugin context, the fork deny and the heddle passage and file entry, including an unmarked file. Pinned, except the gaps in F1 and F2.
- Codex manifest read back from the launch line: `codexengine.TestPrepare_WritesTheManifest`. It pins the same channels, the omitted plugin context with its verdict, and the settings items. Pinned.
- `ListedSkills`, `Plugin` and `Hook` fields: `TestUserSetup_SkillListing` and `TestUserSetup_PluginSessionStartContexts`. Pinned.
- Codex stays silent for a hook listed unrun: `codexengine.TestPrepare_UserSetup`. Judged pinned from the redundancy report; I did not read its body.
- `seatengine.Specs`: `TestSpecs_MatchTheStartedSpecs`. Pinned.
- `NewWithSeams` hands Codex the given hook runner and sandbox probe: `TestNewWithSeams_CodexTakesTheGivenSeams`. The nil-seam fallback is covered by existing tests.
- `registeredConstructors` and `KeepWarm` unchanged: `TestKeepWarm` and `TestNew`. Pinned.
- `Compare` (shared heddle items compared, single-engine items skipped, unrun context counts present, fork deny gated on `RequirementForks` in both directions): `TestCompare/Compare` rows. Pinned.
- Fork-deny tripwire: the `ForkDenyIsAskedOfAnEngineWithForks` row on a fake engine, plus the real-engine `TestSeats` and `TestProducers`, which would fail the day Codex answers `RequirementForks`. Pinned.
- Dropping any single Codex delivery fails the parity test: `TestSeats_CodexManifestHoldsClaudes` and `TestProducers_CodexManifestHoldsClaudes`, compared against a fixture holding every channel. Pinned.
- `PromptItems`, `Unmet`, `UnrunHooks`, `ConfiguredSandbox`, `PrepareEach` (skill expansion, error naming, no manifest): `TestCompare` subtests. Pinned.
- `seatsOf` and `LookupSeat`: `TestCompare/seatsOf` and `TestReport`. Pinned.
- The scan (unlisted `Spec` with `Prompt` fails, no-`Prompt` literal skipped, orch, shuttlecli and testkit skipped, table and seat literals, method glyph, function literal): `TestLaunchSpecLiterals_AreListed`, including the `RealModule` row. Pinned.
- `lym shuttle manifest` output and exit codes (ok, missing item named, unmet never fails, report error): `TestManifestCmd`. The argument count is pinned by `TestCommand_PositionalArgValidation/ManifestNoArgs`. Pinned, except F4 and F5.
- `burlerengine.SeatTables` and `landingshed.FixReviewSeatTable`: only through the catalog in `TestSeats_CodexManifestHoldsClaudes`, which is the public surface. Pinned.
- `report` removes its temp directory: asserted on success rows only. See F3.
- `Install­PassageReader` wrapper in `cmd/lym/main.go`: none. See F6.
- `manifestReportOf` in `internal/shuttlecli/manifest.go`: none. See F5.
- A failing manifest write refuses the launch in either `Prepare`: none. See F7.

**Findings**

F1. Severity MEDIUM, class design. Location: `internal/shuttleengine/claudeengine/manifest.go:345-353`. The branch that lists a failed or unrun plugin hook as an omitted `plugin-context` item has no test. The Claude branch that lists an unrun hook as native also has none: `TestPrepare_WritesTheManifest` asserts only the native `scribe@market/emit` item and `len(Verdict) == 0`. Dropping the `hook.Hook == "" || hook.Reason != hookNotRunReason` handling, or its `DeliveryOmitted` marking, fails nothing. The decision record requires an omitted context to count as missing in the record's verdict. Evidence: `claudeengine/manifest_test.go:224-226`.

F2. Severity LOW, class design. Location: `internal/shuttleengine/claudeengine/manifest.go:332` and `manifest.go:365-368`. Two paths have no row. One is a user-setup resolution that errors, which should log and list no native items. The other is a working directory with no repository root, which should list `CLAUDE.md` alone. The fixture always has a `.git`. Evidence: `claudeengine/manifest_test.go:45-50`.

F3. Severity MEDIUM, class design. Location: `internal/shuttleparity/report.go:43`. The decision record says the verb removes its temp directory on every path including errors. The `defer os.RemoveAll(dir)` is asserted only on success rows. The error rows return before the `os.Stat(reportDir)` check, and `reportDir` is empty for an unknown seat. A `Prepare` error or an unknown-seat refusal that leaks the directory passes. Evidence: `report_test.go:106-117` and `report_test.go:135`.

F4. Severity MEDIUM, class design. Location: `internal/shuttlecli/manifest.go:436-457`. `manifestFields` has three behaviours with no assertion. It strips each manifest's `Verdict`. It lists omitted plugin contexts under `unrun` by engine. It prints `missing` as `channel key` strings. The stub reports in `TestManifestCmd` hold no omitted items, and no row asserts `"unrun"` or the absence of a verdict. The decision record requires a pass with plugin contexts listed unrun, so a regression would still pass. Evidence: `manifest_test.go:23-43` and `manifest_test.go:52-55`.

F5. Severity MEDIUM, class design. Location: `internal/shuttlecli/manifest.go:462-490`. `manifestReportOf` and the `PersistentPreRunE` wiring in `cli.go` have no test. The untested parts are the no-shortname refusal with its way forward, the registry load, the `NewWithSeams` call with `UnrunHooks` and `ConfiguredSandbox`, and the `SeatEnv` assembly. The decision record says the verb runs no hook and no sandbox probe. Evidence: `TestManifestCmd` builds `shuttleCLI` by hand with a stubbed `manifestReport`.

F6. Severity MEDIUM, class design. Location: `cmd/lym/main.go:180-197`. The passage reader that now converts `Passages` and `FileEntries` into a `shuttleengine.Standing` is untested. Every parity or engine test installs its own reader, so dropping `FileEntries` or the conversion there fails nothing, and production heddle file entries would silently vanish from the manifests and from the comparison. Evidence: `seats_test.go:98-110` and `codexengine/manifest_test.go:187`.

F7. Severity LOW, class design. Location: `internal/shuttleengine/claudeengine/claudeengine.go:539-541` and `internal/shuttleengine/codexengine/codexengine.go:757-759`. Both `Prepare` docs say a failed manifest write refuses the launch. No test makes the write fail and asserts the `Prepare` error. The ErrHookUnrun wrapper path also has no test of the way-forward wording.

**Tool calls:** none. My context already held every file I needed.
