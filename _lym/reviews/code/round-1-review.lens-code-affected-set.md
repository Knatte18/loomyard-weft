```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each changed pattern, behaviour and doc claim, find everything outside the diff that reads, asserts or describes it, and report what the diff leaves wrong.

**Affected set per change**

- **`commandPositionPattern` rewrite (shell -c script branch, `do`/`then`/… keywords) and the new `redirectWritePattern` and `rawGoCommandPattern` alternatives.** These reach every row built on them.
  - Renderers `claudeengine/settings.go`, `claudeengine/manifest.go`, `codexengine/guards.go`, `codexengine/command.go` and `configcli` take `row.Patterns` as opaque text. Covered: no change is forced.
  - `claudeengine/testdata/settings-golden.json` is regenerated. Covered.
  - No other fixture holds the pattern text: a grep for the old prefix `env|exec|xargs|sudo|time` finds only rows.go and the golden. Covered.
  - Codex has no golden fixture. Covered: nothing to update.
  - `rows_integration_test.go` has new rows, and `behind_bash_dash_c` flips to true. Covered.
- **Behaviour change: rows now refuse inside a `sh`/`bash -c` script, and `sed -i` after `do`/`then` at top level.**
  - Doc comments in `internal/shellguard/rows.go` (row comments, `rawGoCommandPattern`, `refuseRawPattern`) are updated. Covered.
  - `internal/shellguard/doc.go` is updated. Covered.
  - `internal/shuttleengine/claudeengine/doc.go` is updated only at :681 and :687. Not covered at :706, :723, :733, :734, :747 and :761 (finding 1).
  - Docs in websterengine, planparser and fabricengine that mention `bash -c` describe their own scanners, not the shell guard. Not affected.
- **Change description (`_lym/landing/summary.md`).** It claims the package docs "no longer list `bash -c` as a way past the guard". That claim is false for claudeengine/doc.go (finding 2).
- **PATTERN rules.** `PATTERN-shell-guard-table` still holds: the change stays in `internal/shellguard`'s table, every engine renders every row, and there are no new imports. No declared PATTERN changes exist. Covered.
- **Commit messages.** There are no `Pattern-Departure:` or `Loosened-Check:` trailers and no improvement claims with figures. The flipped `behind_bash_dash_c` row is a behaviour change, not a loosened check. Covered.

**Findings**

1. MEDIUM — consistency — `internal/shuttleengine/claudeengine/doc.go:706`.
   - The per-row guardrail lists in claudeengine's package doc still name `bash -c` as a way past the raw go row (:706), the refuse-raw row (:723), the shell-edit row (:733), the computed-rm row (:747) and the headless claude row (:761).
   - :734 still says "a `sed -i` after `do` or `then`" passes.
   - The diff now refuses all of these; its own test rows `sed_in_place_in_a_shell_script`, `sed_in_place_after_do`, `module_wide_in_a_shell_script`, `in_a_double_quoted_shell_script`, `rm_after_do_in_a_shell_script` and `headless-claude in_a_shell_script` pin it.
   - Fix: change each to "a nested `bash -c`" or "a script file", and drop the `sed -i` after `do`/`then` clause, matching the rows.go comments.
   - Evidence: claudeengine/doc.go:706, :723, :733–734, :747, :761; rows_integration_test.go:205–217, :238, :241, :252, :279.

2. LOW — consistency — `_lym/landing/summary.md` ("Package docs … they no longer list `bash -c` as a way past the guard").
   - The change description's claim does not hold for claudeengine/doc.go, which still lists `bash -c` five times (finding 1).
   - Fix: it resolves with finding 1's fix; otherwise correct the claim.
   - Evidence: summary.md "What changed" paragraph 4; claudeengine/doc.go:706, :723, :733, :747, :761.

**Tool calls**

1. A grep over internal, docs, pattern and stencils for `bash -c`/`sh -c` bypass wording, plus the diff stat. My context lacked the docs outside the diff.
2. Read claudeengine/doc.go:700–765 and grepped codexengine for bypass wording. My context held only the two changed doc lines.
3. A grep for other fixtures holding the old command-prefix pattern and for the `do`/`then` wording. This confirms whether any golden besides the Claude one embeds the patterns.
