```yaml
lens: code-affected-set
outcome: reported
```

The code-affected-set lens: what each change forces outside the lines it touches.

**Per change, the set found and whether the diff covers it**

1. **`inPlaceEditCommandPattern`, now reading past closed heredocs (`rows.go:329-349`), through `shellEditPattern`.**
   - `internal/shuttleengine/claudeengine/testdata/settings-golden.json` (`TestBuildSettings_MatchesGolden`, `settings_test.go:1150-1200`): covered. It was re-rendered in commit 56a99270b, and the rendered hook I pulled from it holds the new alternation.
   - Codex rendering (`internal/shuttleengine/codexengine/guards.go:96-111`): covered. It builds `grep -Eq` from `row.Patterns` generically and has no golden that needs updating.
   - Its tests in `guards_run_test.go:80-81` (shell-edit `sed -i` / `sed -n`) are unaffected.
   - Other shellguard consumers (`configcli`, `claudeengine` manifest/settings, `orchcli` denypython test, `cmd/lym/norecoverydeny_test.go`): unaffected. They read `Forms`, `Steer` and `Notice`, and none of those changed.
   - Steer and notice text (`rows.go:103-112`): unchanged, and still accurate. "sed -i, perl -i … are refused here" now also holds after a closed heredoc.
   - `internal/fabricengine/refscanner_test.go:57` ("command after a heredoc is still checked"): a separate scanner, not affected.
2. **The `inPlaceEditCommandIn` extraction (a func replacing the closure).** The only callers are `rows.go:348-349`. Covered.
3. **New `heredocBodyLinePattern`, `closedHeredocPattern` and `afterClosedHeredocScope`.** No other reader. Covered.
4. **`doc.go` package doc on the heredoc reach (`doc.go:22-26`).**
   - Covered at lines 22-26.
   - Left stale at line 15: "one unit at a time: … and at most one heredoc operator plus its line". The shell-edit in-place match now reads any number of closed heredocs and the lines after them. Not covered (finding 1).
   - The claim at `doc.go:18-19` ("The script of a … -c in command position is read the same way, one level deep. A guard thus refuses inside the script what it refuses outside it") no longer holds on lines after a closed heredoc. The new scope has no shell -c reach (`rows.go:331`), and `doc.go:22-26` does not state the exception. Not covered (finding 2).
5. **Shell-edit row comment (`rows.go:835-838`).** Covered. `commandPositionPattern`'s comment (`rows.go:192`, "sees nothing after a heredoc operator's line") is still true for that prefix.
6. **Rule documents.**
   - `PATTERN.md:49` `PATTERN-shell-guard-table`: the row stays in shellguard's table, and every engine renders it through the generic pattern loop. Complies. It has no background file.
   - `pattern/PATTERN-leaf-packages.md` (shellguard's import set): no new imports. Complies.
   - `PATTERN-test-speed`: rows were added to the existing tier-1 `TestRows_PatternsDenyAndAllow`. Complies.
   - No declared PATTERN changes, and none required.
7. **Commit messages.**
   - 91c8b0bda carries a `Root-Cause:` naming `topLevelReachPattern` stopping at the heredoc operator's line. That matches the code at `rows.go:134` and the fix.
   - No `Pattern-Departure:` trailer, and none needed.
   - No `Loosened-Check:` trailer needed: the diff adds rows only, and the golden is re-rendered, not loosened.

**Findings**

1. **LOW, consistency, `internal/shellguard/doc.go:14-15`.** The package doc still says a command-shaped row reads "at most one heredoc operator plus its line". The shell-edit in-place match now reads past one or more closed heredocs (`rows.go:331`, `(shellUnit* closedHeredoc)+`). Lines 22-23 add the exception, but line 15's general statement now contradicts it for this row. Evidence: `doc.go:14-15` against `rows.go:326-333`.
2. **LOW, consistency, `internal/shellguard/doc.go:18-19,22-26`.** The doc says a shell -c script in command position is read the same way, so a guard refuses inside the script what it refuses outside it. That is untrue on the lines after a closed heredoc: the new scope's position has no shell-script reach (`rows.go:331`). As a result, `cat <<EOF\ny z\nEOF\nbash -c 'sed -i s/a/b/ f'` passes (my runs: `heredoc_then_sh_c_edit`, `heredoc_then_dq_sh_c_edit`), while `bash -c 'sed -i s/a/b/ f'` alone is denied. Neither `doc.go:22-26` nor the row comment at `rows.go:835` names this exception. Evidence: `doc.go:18-19`, `rows.go:331`, `.scratch/rv/cases.txt` run results.

**Tool calls**
- Grep of `.md`, `.go` and `.yaml` for heredoc-guard prose, plus the PATTERN/shellguard mentions: to find docs and rules outside shellguard that describe the changed behaviour, which my context lacked.
- Grep for packages importing `shellguard.`, a codexengine testdata listing and `PATTERN-leaf-packages` lines 40-70: to find the consumers and the codex goldens, and to check shellguard's import rule.
- Read of `codexengine/guards.go` pattern rendering and grep of shell-edit/heredoc rows in the other engine tests: to confirm how Codex renders the changed pattern and whether any other fixture asserts it.
