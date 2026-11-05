```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff (`settings.go` attribution change and its test edits).

**List: function or branch, case run, verdict**

- `attributionSettings` (`settings.go:190-195`): the zero value marshals as `{"commit":"","pr":""}`. Neither field has `omitempty`, and the `pr` tag matches Claude Code's key. Correct.
- `settingsDoc.Attribution` (`settings.go:228-232`) and `buildSettings` (`settings.go:402`): the doc literal never sets `Attribution`, so it stays zero. `json.Marshal` emits `"attribution":{"commit":"","pr":""}` after `promptSuggestionEnabled`. I ran four inputs:
  - standalone, `creelFlagDirPosix == ""`;
  - in a hub, `creelFlagDirPosix != ""`, where only the Stop and PostToolUse hooks are replaced and the doc is never rebuilt;
  - an empty `contextAfterCompaction`;
  - a non-empty `contextAfterCompaction`.

  All four paths marshal the same top-level struct, so the key is always present. Correct.
- `openSettingsDoc` and `buildOpenSettings` (`settings.go:197-222`): the doc is built with `Hooks` only, so `Attribution` is zero and marshals to `{"hooks":{...},"attribution":{"commit":"","pr":""}}`. Both fields are unconditional. The error path is unchanged, since `json.Marshal` of plain structs cannot fail. Correct.
- `writeOpenFiles` (`command.go:467`) and `OpenLaunch` (`command.go:416`): `writeOpenFiles` writes the `buildOpenSettings` bytes verbatim, and the diff changes nothing in either function. With an empty `creelFlagDirForHook` no `--settings` is passed and no file is written, so that session keeps the worktree's own attribution. This is documented in the summary's Limits and is not a regression. Correct.
- `checkOpenCreelFiles` (`command_test.go:558-582`):
  - The `keys` map must hold exactly 2 entries (`hooks` and `attribution`), and `doc.Hooks` must hold exactly 2 event keys. This still pins that no other top-level key or hook event appears.
  - `Attribution` decodes into `map[string]string`. A missing key would give nil and fail `DeepEqual`, and extra keys would fail too.
  - The hook command assertions after it are unchanged. Correct.
- `TestSettings_CreditNoSession` (`settings_test.go:76-105`): `parseSettings` decodes to `map[string]any`, so the got value is `map[string]any{"commit":"","pr":""}`, which matches the wanted shape under `reflect.DeepEqual`. A missing key gives nil and fails. The subtests close over `tt` safely, since `go.mod` is at Go 1.22 or later per the golang-testing skill. Correct.
- `settings-golden.json`: I did not inspect each of the 16 changed entries line by line. `TestBuildSettings_MatchesGolden` compares byte for byte, so a wrong entry would fail that test rather than pass silently.

**Findings**

1. LOW, class `consistency`, `internal/shuttleengine/claudeengine/command.go:404` and `:464`.
   - The `OpenLaunch` doc says the opened session's `settings.json` holds "creel's hook delivery alone".
   - The `writeOpenFiles` doc says it holds "creel's hook delivery over the flag directory".
   - After this diff the file also carries `attribution`, so the `OpenLaunch` doc (and the narrower `writeOpenFiles` one) states something the code no longer does.
   - The diff updated `doc.go`, `settings.go`, `docs/overview.md` and the test helper comment, but not these two comments. Evidence: `command.go:404`, `settings.go:197-203`.

No `design` findings: I found no incorrect result, swallowed error or inconsistent failure state in the changed code.

**Tool calls:** none. The manifest already held every definition, caller and test this lens needed.
