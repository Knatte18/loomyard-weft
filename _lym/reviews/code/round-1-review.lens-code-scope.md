```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced the diff's changes to the Goal's points. The decision record has no `Added after Discussion` entries.

**Decisions (Goal points):**
- **G1.** `internal/shellguard` parses the script of a shell -c call like a top-level command.
- **G2.** It applies the same refusals to that script, so the agent gets the row's way forward and no dialog stalls the run. The Seen command is a computed-path `rm` inside a `sh -c` loop.

**Changes and what each traces to:**
- **C1.** The script branch of `commandPositionPattern`: `commandSeparatorPattern`, `topLevelReachPattern`, `commandWordPrefixPattern`, `shellDashCPattern`, `singleQuotedScriptUnitPattern`, `doubleQuotedScriptUnitPattern`, `shellScriptReachPattern` (rows.go hunk @@ -120). Traces to G1 and G2.
- **C2.** The keywords `do|then|else|if|elif|while|until` added to `commandWordPrefixPattern`. They apply at the top level as well as in the script (rows.go diff line 37). The in-script half partly traces to G1, since the Seen command is a loop. The top-level widening traces to no decision (finding S1).
- **C3.** The script branch of `redirectWritePattern` (diff lines 84–86). Traces to G2, the shell-edit row.
- **C4.** The fifth alternative of `rawGoCommandPattern`, a lym environment-variable name in a script (diff lines 112–113). Traces to G2, the raw-go row.
- **C5.** `'` added to the word-end class of the go test/build/vet `...|all` alternative (diff line 107). Forced by C1, because a script's closing quote ends the word.
- **C6.** Row comments in `rows.go` drop `bash -c` from their pass lists and are re-broken one sentence per line. The shell-edit comment also drops "a sed -i after do or then". Forced by C1 and C2, which made those statements stale, and by the comment lint on the touched lines.
- **C7.** `internal/shellguard/doc.go` and `internal/shuttleengine/claudeengine/doc.go`. Forced doc updates for C1 and C2.
- **C8.** `claudeengine/testdata/settings-golden.json`. Forced: it is the rendered patterns, and the notices are unchanged.
- **C9.** New and changed rows in `rows_integration_test.go`. Forced test coverage of C1–C5. The flip of `behind_bash_dash_c` follows from G2.

**Decisions and the changes that build them:**
- **G1:** C1, plus C2's in-script half.
- **G2:** C1, C3 and C4. They are partial for the raw-go row's double-quoted expansion (finding S2).

**Findings:**

- **S1.** MEDIUM, scope.
  - **Location:** `internal/shellguard/rows.go`, `commandWordPrefixPattern` (diff piece 001, line 37).
  - **Summary:** the shell keywords are admitted at the top level too, which widens every command-shaped row's top-level denies beyond the Goal. Examples: `for f in a b; do sed -i … $f; done` and `for f in a b; do rm $f; done` are now refused.
  - **Why it is not forced:** the Seen command's `rm` follows `&` inside the loop (test row `background_rm_in_a_shell_script_loop`, piece 001 line 251), so the keywords are not needed to refuse it.
  - **Evidence of no license:** `_lym/landing/summary.md` "Design decisions" says the keywords go "past the record's idea". No `Pattern-Departure:` trailer and no Added-after-Discussion entry licenses it (git log 7346d6a8..HEAD).
  - **Fix:** leave the code in place and defer to the run's parent, who records the widening with `lym loom decision add` or refuses it. The fixer cannot settle the license.
- **S2.** LOW, scope.
  - **Location:** `internal/shellguard/rows.go`, `rawGoCommandPattern`'s fifth alternative (diff piece 001, lines 112–113).
  - **Summary:** G2's "same refusals" is only partly built for the raw-go row. At the top level, a lym variable expanded inside a double-quoted string is refused by the fourth alternative, as in `echo "$LYM_GATE_SLOT"`. Inside a single-quoted script, as in `sh -c 'echo "$LYM_GATE_SLOT"'`, nothing reaches it.
    - `singleQuotedScriptUnitPattern` reads the `\"…\"` string as one unit.
    - Its boundary class `\\[^"]` rejects the JSON-escaped quote before `$`.
    - The fourth alternative's `'[^']*'` unit swallows the whole script.
  - **Evidence:** the doc comment states the narrower behaviour, "outside a quoted string of the script" (piece 001, line 99). I reasoned this from the regex text and did not run it.
  - **Fix:** either add a script-scoped double-quoted-expansion alternative mirroring the fourth, or have the parent accept the documented gap.

Tool calls: none. Everything came from the inherited context: the diff pieces, the decision record, `summary.md`, the git log and the `rows.go` definitions.
