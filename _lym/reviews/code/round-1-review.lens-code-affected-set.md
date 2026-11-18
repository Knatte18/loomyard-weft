```yaml
lens: code-affected-set
outcome: reported
```

The steer-text change leaves nothing outside the touched lines stale. I found no findings.

**What the diff changes:** every row's plain `Steer` and every non-empty engine `EngineTexts[...].Steer` that `shellguard.Rows` returns now opens with the `wholeCommandRefusal` clause. `Notice` is unchanged.

Per change, the affected set I found:

1. **Plain `Row.Steer` consumers:**
   - Claude's deny echo goes through `claudeengine/settings.go:357` (`renderShellGuard`, which wraps `denyJSON(row.Steer)`) and `settings.go:346`. Both are rendered from `Rows`, so they pick the clause up automatically. Covered.
   - The `standingDeny.steer` forbidden-char panic at `settings.go:569` is covered, since the clause holds no quote or backslash.
   - `manifest.go:296-299` (`claudeKeys` keyed by steer) maps only the Claude-own denies `steerWebsterForkDeny` and `steerAskUserQuestionDeny`. The shellguard rows are keyed by `row.Name` (`manifest.go:301-302`), so they are unaffected.
2. **Engine steers (`SteerFor`):**
   - Codex reaches them through `codexengine/guards.go` (`guardCommand` → `denyDecision(row.SteerFor(engineName))`), and the Codex steer now carries the clause. Covered.
   - The clause holds no `"` or `\`, so `denyDecision`'s unescaped JSON stays valid, and the `posix.Quote` wrapping is unaffected.
3. **Tests that assert steer text:**
   - `claudeengine/settings_test.go:480-487, 537, 547, 640-650, 1014`, `hookcmd_integration_test.go:493`, `codexengine/guards_test.go:79` and `guards_run_test.go:93` all derive the expected text from `Rows`/`shellRow`, so none is stale.
   - `shellguard/rows_test.go` `TestCheckTexts` and `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` are updated in the diff.
4. **Fixtures:**
   - `claudeengine/testdata/settings-golden.json` is the only fixture holding steer text. It is updated, and it equals the base byte for byte once the clause is stripped (checked in my context).
   - No Codex golden holds steer text. Covered.
5. **Notices:** these are unchanged, so `codexengine/guards.go` `denyNotice`, Claude's `buildDenyNotice` and the system-prompt notices need nothing. The raw-go notice stays as it was (`rawGoSteer + "."` at `rows.go:494` is built before the prefix). Covered.
6. **`Forms`:** unchanged, so `cmd/lym/norecoverydeny_test.go` (`shellGuardForms`) is unaffected. The clause names no denied form, and shellguard lies outside the scanned bound modules anyway.
7. **Docs, stencils and specs that describe steers:**
   - `contracts/stencils/engine/{claude,codex}/edit-directive.md:14`, `docs/dotnet-hub.md:173`, `docs/overview.md:405,717` and `contracts/stencils/retro/retro-template-task.md:52` describe what the guard refuses, not how a steer opens. None is made wrong.
   - `contracts/specs/refusal-spec.md` binds webster, shed and loom only.
   - The quotes in `docs/research/retro/...` are a measurement report, which is exempt.
8. **PATTERN rules:**
   - The `PATTERN-shell-guard-table` text ("only `internal/shellguard`'s table, every engine rendering every row") is upheld: the clause is applied inside the table's `Rows`, and both engines render it.
   - `PATTERN-refusal-way-forward` and its No-denied-recovery rule: the clause names no denied form.
   - The directive declares no PATTERN changes, so nothing is owed under "Declared PATTERN changes landed".
9. **Commit messages:** a8a2131c, 315046fb and b837f1e0 carry no `Pattern-Departure:` trailer, and none is needed. a8a2131c's body matches the diff. The two `fix:` commits only rewrap comments, which matches their subjects.

Tool calls:
1. Grepped claudeengine for `.steer` consumers, fixtures holding steer text and tests that assert `Steer`/`SteerFor`, because the manifest did not cover test assertions or fixtures outside the golden.
2. Read `claudeengine/manifest.go:280-320` (`guardCandidates`/`claudeKeys`), because a map keyed by steer text could break when the text changes.
3. Grepped `contracts`, `docs` and `plugins` for files that mention the shell guard, to find prose that describes steers.
4. Read those mentions in the stencils and docs, to confirm none describes how a steer opens.
