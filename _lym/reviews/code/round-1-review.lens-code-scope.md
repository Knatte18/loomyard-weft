```yaml
lens: code-scope
outcome: reported
```

Scope trace for the code-scope lens: each change in the diff mapped to a decision, and each decision to the changes that build it.

**Decisions** (the `## Goal` points and the one Added-after-Discussion entry)
- **D-Aim**: cut the tier1 time of `internal/webstercli` and `internal/loomcli`.
- **D-W**: in `internal/webstercli`, pass the environment and working-directory values in, then mark the tests parallel; `TestResetCmd` is the longest.
- **D-L**: in `internal/loomcli`, share one built hub copy between the tests that only read it, and fold rows of the same shape.
- **D-K**: keep every assertion; drop none.
- **D-M**: measure each package's tier1 time before and after, twice each, and put both in the change description.
- **A1** (parent, Added after Discussion): the writer may run the timing gate itself and records the load average at each start.

**Per change**
- **C1**: the `websterCLI.syncOptions` and `strandName` fields, the `fabricSync` options argument, every verb's call site, `masterStrandLookup`, `reset`'s `ResetPairCode` options, and the orphaned-Master guard reading `c.strandName` (`cli.go`, `sync.go`, `wiring.go`, `batchpause.go`, `ownership.go`, `reset.go`, the verb files). Traces to D-W.
- **C2**: webstercli tests drop `t.Setenv` for the skip switches and the strand name, use `newSkipGitVerbsFixture` or the fields, and call `t.Parallel`. Traces to D-W.
- **C3**: webstercli tests that set no environment but gain `t.Parallel`, such as `TestCommand_LongStringsHaveNoStaleBatchLanguage`, the `TestValidateCmd_*` tests, refusalnote, `TestWebsterCLI_PairSlug`'s subtests and `TestPersistentPreRunE_PlanDirAnchoredAtSubpath`. Traces to D-W ("mark the tests parallel") and D-Aim.
- **C4**: the `TestVerifyVerb_*` tests fold into `TestVerifyVerb`, and `TestGateParity_VerifyGate` sets `PATH` once and runs its cases in parallel. Traces to D-W (`PATH` is environment).
- **C5**: `TestResetCmd` gives each case its own hub copy and runs the cases in parallel, with `newResetFixture(syncOptions)`. Traces to D-W, which names it.
- **C6**: loomcli tests move to `pairHubShape` / `hubforge.TemplatePairSlug`, covering `ownercheck`, `peers`, `landingdeps`, `subjects` and the rest. Traces to D-L as the writer reads it: one built pair template, copied per test.
- **C7**: rows that shared one copy (`TestApproveDiscussion_Real`, `TestSeedDiscussedRecord_Real` and both commitstatus tests) now take a copy each and run as parallel subtests. Forced by C6: the template holds only one slug, so rows can no longer add separate pairs to one copy.
- **C8**: loomcli serial tests with no global state gain `t.Parallel`: circling, carryover, `TestCommitStatusSeam_Real`, describe-table, drift-verdict, rework and parentbranch. Traces to D-Aim; the loomcli bullet of D-L does not ask for it.
- **C9**: `TestWireLanding_TellsLanguageSkills` and `TestWireLanding_FillsSamplerSessionSeams` fold into `TestWireLanding_FillsTheLandingSeamsPerRecipe`. Traces to D-L ("fold rows of the same shape") and keeps both tests' assertions (D-K).
- **C10**: the `seedLoomConfig` doc comment changes from "creates &lt;anchorPath&gt;" to "creates&lt;anchorPath&gt;" (`internal/loomcli/hubfixture_helper_test.go:373`). Traces to none.
- **C11**: commit 5b82f60 splits two over-long comments. Forced by C6/C5.
- **C12**: the `## Measurements` table in `_lym/landing/summary.md`. Traces to D-M and A1.

**Per decision**
- **D-Aim**: C1 through C9.
- **D-W**: C1 through C5, partly built (finding F1).
- **D-L**: C6, C7 and C9. The "share one built hub copy" half is built only as copy-per-test (finding F2).
- **D-K**: C2, C4, C5, C7 and C9 keep their assertions as far as this lens reads them; the code-tests lens owns the full comparison.
- **D-M**: C12.
- **A1**: C12. It permits a `--gate darn-write --tier 1` call; the summary ran without `--gate` and says why. That is a "may" permission, so no finding.

**Findings**

1. **F1. MEDIUM, scope.** In `internal/webstercli`, D-W is only partly built: a set of webstercli tests still set the environment or the working directory and stay serial.
   - The serial tests:
     - every wiring test in `wiring_test.go`, such as `TestWire_ModeHubSelectsHubMode` and `TestWireHub_LanguageSkills`;
     - `TestWire_HubLoadsSecretPathsFromFabricYAML`;
     - `TestCommand_WronglyEnteredHub_Refuses` and `TestCommand_StandalonePreRun_TargetDirectoryUnchanged`;
     - `TestCommand_StandalonePreRun_ReachesRunsOwnValidationGate`;
     - `TestCommand_GroupGuard_OutsideGitRepo` (`t.Chdir`).
   - The entry asks to pass the environment and working-directory values in, then mark the tests parallel. The summary leaves these out of scope ("production reads those through code this change leaves alone"), and no Added-after-Discussion entry records that narrowing.
   - Evidence: `internal/webstercli/wiring_test.go:15` and `:207`, `internal/webstercli/cli_integration_test.go:51-53` and `:135`, `internal/webstercli/cli_test.go:41-44`, `_lym/landing/summary.md:22`, `_lym/discussion/decision-record.md:10-11`.

2. **F2. LOW, scope.** In `internal/loomcli`, the "share one built hub copy between the tests that only read it" ask is met only by copying a template that is built once per binary. No test that only reads takes `hubforge.SharedHub`.
   - Every moved test still calls `CopyHub`. For example, `TestPairSlugOf` only resolves locations and calls `pairSlugOf`, yet takes its own copy.
   - The summary heading "one built pair shared through the template" states the narrower reading as if it were the ask.
   - Evidence: `internal/loomcli/gatefixer_integration_test.go:14-44` (`CopyHub` in a test that only reads), a grep finding no `SharedHub` in `internal/loomcli`, `internal/hubforge/template.go:5` (`SharedHub` exists for tests that only read), `_lym/discussion/decision-record.md:12`.

3. **F3. LOW, scope.** C10 is an edit no decision asks for: it deletes the space in the `seedLoomConfig` doc comment, leaving "creates&lt;anchorPath&gt;/_lym/config/loom.yaml".
   - Evidence: `internal/loomcli/hubfixture_helper_test.go:373` (diff 001, line 373).

Tool calls:
- `grep SharedHub` over loomcli and webstercli, plus a read of `TestPairSlugOf`: my context did not show whether any loomcli test only reads a hub, or whether `SharedHub` is used.
- An awk listing of webstercli top-level tests without `t.Parallel`: my context held only the diff, not the full set of tests still serial.
