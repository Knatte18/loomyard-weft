Lens fix-code-correctness: I ran each changed function and added test through concrete inputs against the code as it stands in context.

**Findings**

1. **MEDIUM — `internal/shuttlecli/manifest_test.go`, `TestManifestReportOf_RefusesAHubWithNoShortname` (F7).**
   - The fix tests only the no-shortname refusal.
   - F7's main premise is that swapping `shuttleengines.NewWithSeams(... UnrunHooks, ConfiguredSandbox)` for one that runs real hooks and the sandbox probe passes every test.
   - Evidence: the refusal returns at `manifest.go:118`, before `NewWithSeams` at line 124 is reached.
   - Dropping the seams therefore still fails nothing.
   - The fixer should list this under the report's deferred section with the reason, or add a row that reaches `NewWithSeams`.

2. **LOW — `cmd/lym/passagereader_test.go` (F8).**
   - The file entry is keyed `agents-file`, so its expected `Engine` is `""`.
   - The test therefore never pins an engine key on a file entry.
   - Dropping the conversion would still fail, because the file entry would go missing.
   - A change that blanked `Engine` on file entries would pass, but the prompt row does pin the engine key.
   - Not blocking.

**Checked, no defect**

- **`setupCandidates` (F3):**
  - A hook with empty `Plugin` now gets key `hook.Name`, the same key as Claude's `hook.Name` branch (`claudeengine/manifest.go:346-349`), so the Verdict agrees.
  - Input `{Name:"/plugins/p/hooks/hooks.json", Reason:"unreadable"}` gives a `plugin-context` item keyed by that path, with delivery `omitted`, listed after `q@m/slow`.
  - The Verdict order in the test therefore matches.
  - The new Codex row would fail with the old `continue` in place.
- **Claude rows (F10):**
  - Writing `not json` to the hooks file makes `sessionStartHooks` return a parse error.
  - `Name` is then `filepath.Join(root, hooks, hooks.json)`, which equals `f.hooksFile`, so the omitted item and the Verdict key match.
  - The unreadable-global row turns the global file into a directory, so `instructionFiles` errors, `resolvedCandidates` logs and returns no native items, and the launch still succeeds.
  - The Verdict assertion (`slices.Equal` of nil with nil) holds for rows that omit nothing.
- **Blocked-manifest rows (F11):**
  - The planted `manifest.json` directory makes `state.WriteJSON` fail.
  - Claude now appends `way forward: make the run directory writable` and Codex already did, so both assertions on "way forward" hold.
  - Each row fails if the write error is swallowed.
- **`TestReport_RemovesTheTemporaryDirectoryOnARefusal` (F5):**
  - Removing `defer os.RemoveAll(dir)` leaves a `manifest-report-*` directory, so `after` would exceed `before` and the test fails.
  - The test is serial, so no parallel sibling adds directories during the glob.
  - The row with the failing prepare function reaches `PrepareEach`.
  - The unknown-seat row refuses before any prepare runs.
- **`manifestFields` assertions (F6):**
  - The `unrun` row prints `"unrun":{"codex":["plugin-context p@m/emit"]}` and `"verdict":null`.
  - Dropping `manifest.Verdict = nil` would print `"verdict":[`, which the `wantAbsent` check catches.
- **Constants and callers (F1, F2):**
  - The `target.go` row values are byte-identical to the old literals.
  - `capabilities.go` still returns `["CLAUDE.md","CLAUDE.local.md"]`, so `capabilities_test.go` holds.
  - `agentsOverrideFile` is still an untyped string constant, so its other uses compile.
- **`readHeddleStanding` (F8):**
  - It is a straight move of the old closure.
  - `main.go` passes it to `InstallPassageReader` with the same signature.
  - The gate run of `./cmd/lym` passed.

**Further tool calls:** none; the context already held every file and the gate output.
