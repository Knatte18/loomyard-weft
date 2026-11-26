```yaml
lens: code-affected-set
outcome: reported
```

I reviewed what the diff forces outside its own lines, under the code-affected-set lens.

**Per change, the affected set and whether it is covered**

1. **`-update` flag and the package-level `settingsGolden` type in `internal/shuttleengine/claudeengine/settings_test.go`.**
   - Covered: the only `flag` definition in the package is the new one, so it clashes with no other test flag.
   - Covered: the golden is the only reader of `testdata/settings-golden.json`.
   - Covered: `lym gate test` forwards the words after `--` (`internal/gatecli/cli.go:108`) and runs the tier command in the tree root (`tierrun.go:526`), so the file is written into the worktree.
   - Covered: a `-run` filter prevents a carry skip (`gatecli/doc.go:237`).
   - Covered: the codex engine has no golden to match.
   - Covered: no stencil or spec describes a throwaway renderer that would now be stale.
   - Covered: no PATTERN entry or background file covers goldens or `-update` (grep of `pattern/` and `PATTERN.md`).
   - Not covered: the golden path literal is now repeated (finding 2).
2. **`internal/shellguard/doc.go`, new paragraph on writing a row exception.** The reader is the next row author, inside this package. The rule set is `PATTERN-shell-guard-table` and `PATTERN-code-conventions`. No other doc or stencil states row exceptions (grep for `unquotedWordOtherThan`, "must not match" and "states an exception" hits only `rows.go` and `doc.go`). Covered.
3. **`internal/shellguard/doc.go`, new paragraph naming the Claude engine's golden and its gate command.** Not covered: it documents another package (finding 1).
4. **`Row.Patterns` comment and `unquotedWordOtherThan` doc in `rows.go`.** The only caller is `sessionBusCommandPattern` (`rows.go:424`), which the doc names as its example. Covered.
5. **New session-bus test row.** This is test-only, and nothing outside the test reads it. Covered.
6. **Commit messages of the range.** There are no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none is needed: the compare path loosens no check and the task asks for no bug fix. No declared PATTERN changes are listed, so none were checked. Stencil wording does not apply because no stencil changed. Covered.

**Findings**

1. **BLOCKING, class `design`, location `internal/shellguard/doc.go:38-39`.** The `doc.go` of `internal/shellguard` now documents a test and gate command of another package. That breaks `PATTERN-code-conventions` ("a package's `doc.go` documents only that package").
   - Evidence: `pattern/PATTERN-code-conventions.md`, "Go documentation" rule ("it documents the package; a rule other code must follow is a PATTERN entry, never a `doc.go` paragraph"), and diff `001` lines 19-20.
   - The paragraph names "the Claude engine's settings golden" and `lym gate test ./internal/shuttleengine/claudeengine -- -run TestBuildSettings_MatchesGolden -update`. That command already sits beside the flag at `settings_test.go:1150-1152`.
   - Fix: drop the paragraph from shellguard's `doc.go`, or state it through a channel the rule allows.
2. **BLOCKING, class `consistency`, location `internal/shuttleengine/claudeengine/settings_test.go:1153` and `:1168`.** The golden path `testdata/settings-golden.json` is now written twice: as a literal inside the flag's usage string, and again as `goldenPath := filepath.Join("testdata", "settings-golden.json")`. The gate command is also repeated verbatim in `settings_test.go:1152` and `shellguard/doc.go:39`.
   - `PATTERN-code-conventions`, "Named constants", requires "one named constant, declared once and referenced everywhere, never a repeated literal" for a name or path that can go stale.
   - Evidence: diff `002` lines 20-21 and 48, and diff `001` line 20.
   - This is test code, so the handler may weigh severity; the constraint text makes every violation BLOCKING.
   - Fix: declare the path once as a constant and build both the usage string and `goldenPath` from it.

**Tool calls**

- grep of md, go and tmpl files for "settings golden", "renderer" and the exception wording: to find docs or stencils outside the diff that describe golden regeneration or row exceptions, which my context did not cover.
- `ls` of `codexengine/testdata`, grep for "golden" in `shuttleengine` and `shellguard`, and the read of `pattern/PATTERN-code-conventions.md`: to check for a second engine golden and to read the background of a PATTERN entry the diff touches, which the manifest had not read.
- grep of `pattern/` and `PATTERN.md` for golden, `-update` and testdata: to find any repository rule on goldens or update flags.
