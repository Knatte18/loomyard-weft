Lens fix-proposal: challenge each reviewer proposal the fixer applied as proposed.

**Findings**

1. **LOW.** Changed text: F7, `internal/shuttlecli/manifest_test.go`, `TestManifestReportOf_RefusesAHubWithNoShortname`.
   - The reviewer's proposal had two rows: a positive row asserting a seat's report holds both engines' manifests without the hook runner or sandbox probe, and the no-shortname refusal.
   - Only the refusal row exists.
   - The `NewWithSeams(... UnrunHooks, ConfiguredSandbox)` wiring is still unpinned. Replacing it with real hooks would still pass every test.
   - The fixer's report must disposition F7 as partly deferred or `fixed-differently` with a reason. It should not claim it as `fixed`.
   - Evidence: the new test calls `manifestReportOf` with a hub that has no shortname, and the call returns before `shuttleengines.NewWithSeams` runs.

2. **LOW.** Changed text: F5, `internal/shuttleparity/report_test.go`.
   - The reviewer asked for the removal assertion on every row of `TestReport`. The fixer added a separate serial test `TestReport_RemovesTheTemporaryDirectoryOnARefusal` and left `TestReport` unchanged.
   - This is a different change from the one proposed, so it should be marked `fixed-differently` with a reason.
   - It also holds up on correctness. `lymtemp.Dir()` is the process's own run directory, so other test binaries cannot disturb the glob.
   - `t.Parallel()` tests pause until the serial tests finish, so the count comparison is deterministic.

3. **LOW.** Changed text: F11 and the `way forward: make the run directory writable` suffix added to the Claude manifest-write error.
   - The suffix is applied unconditionally. If `state.WriteJSON` already puts a way forward in its own error, the message now carries two.
   - I could not confirm this. My grep of `internal/state` for `way forward` in `WriteJSON` found nothing, so a duplicate is unlikely.
   - No action is needed beyond noting it.

**Proposals checked, no defect found**

- **F1, F2:** The new constants are unexported in `heddleengine`, and the exported file-name constants are imported by `claudeengine` and `codexengine`. There is no import cycle, because `claudeengine` and `codexengine` already imported `heddleengine`.
- **F3:** The Codex omitted-hook key now matches Claude's for an empty `Plugin`, and the new row asserts the verdict.
  - Claude keys an errored hook with a plugin by `hook.Name`, while Codex keys it `Plugin/Hook`. This asymmetry predates the fix and is not caused by it.
  - The verb runs under `UnrunHooks`, so every hook there has the reason `not run` and the keys agree.
- **F9:** The Claude comment and log text now claim that a Codex launch refuses on the same source error. Codex resolves the setup through `c.resolveUserSetup` at `codexengine.go:204`, which supports the claim.
- **F8:** The `agents-file` target in the test is correct. A `claude-file` entry is normalised to `agents-file` on read, which is why the first run reported an empty engine key.
- **F12:** The `//testtiming:keep` directive sits after a blank `//` at the end of the doc comment, the form gofmt keeps for a directive. The gate passed.

**Calls made**
1. `grep` on `codexengine.go` for `userSetup` and `UserSetup(`: to confirm Codex's own setup resolution, which my context lacked.
2. `grep` on `internal/state` for a `way forward` in `WriteJSON`: to check whether the F11 suffix duplicates one.
