```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each branch the diff adds or changes, using the rendered hook from the golden at HEAD and at base `4aa85ab3a` through `.scratch/rv/run.sh`.

**What I ran, branch by branch**

- `heredocBodyLinePattern` (`rows.go:317`):
  - A body line holding `\"` or `\\\\` stays inside the command string: correct.
  - A JSON-escaped `\"` inside a body line is crossed by `\\[^n]`, so a "body" can run on into a later double-quoted string. That is wrong in combination with the closing-line rule (finding 1).
- `closedHeredocPattern` (`rows.go:324`):
  - Empty body, `<<'EOF'`, `<<"EOF"`, `<<-` with a tab-indented close, two heredocs in sequence: all denied correctly.
  - The real closing line ends the command: allowed, correct.
  - A body with a bare-word line before an edit spelling: wrong (finding 1).
  - Delimiter `END-X` or `EOF.`: wrong (finding 2).
  - Two heredocs on one line (`cat <<A <<B`): wrong (finding 2).
  - Here-string `<<<abc`, arithmetic `1<<2`: wrong (finding 4).
- `afterClosedHeredocScope.position` (`rows.go:329-333`):
  - Blank line, indented line, comment line, `cd y &&`, `do`, `| xargs`, `$( … )` and `( … )` around the heredoc: all denied correctly.
  - A shell `-c` script after the closing line or around the heredoc: wrong (finding 3).
- `inPlaceEditCommandIn` (`rows.go:339-344`): the existing scopes still deny every pre-existing shell-edit row, including the `bash -c 'sed -i …'` baseline. Correct.
- `inPlaceEditCommandPattern` alternation (`rows.go:348-349`):
  - The entry's exact command shape is denied (it was allowed at base): correct.
  - `sed -n` after a heredoc is allowed: correct.
  - The redirect after a heredoc is unchanged (still passes, as documented): correct.
- Cost: a 110 KB command with five 400-line heredocs ran in 32 ms with no match and 64 ms with an edit at the end, under GNU grep. The hook string grew from 4592 to 5603 bytes. No problem.

**Findings**

1. **MEDIUM, design**, `internal/shellguard/rows.go:324`. The heredoc body is not skipped when it holds any bare-word line. A later body line, or quoted text after such a line, that starts with an edit spelling is now refused even though it edits nothing. This contradicts the entry's "while still skipping the heredoc body".
   - Commands allowed at base and denied at HEAD:
     - `cat > .scratch/msg.txt <<'EOF'\nSummary\n\nsed -i is now refused…\nEOF\ngit commit -F …`
     - `cat > .scratch/notes.txt <<EOF\nFindings\nperl -pi slipped…\nEOF`
     - `cat > .scratch/fix <<'EOF'\n…\ndone\nsed -i s/c/d/ g\nEOF\nls`
     - `x=$((1<<2))\necho "a\nb\nsed -i x"`
   - The false denial is documented in `rows.go:837-838`. Prose that names the command, such as a commit message or notes, is an ordinary agent output.
   - Evidence: the `cases2.txt` and `cases.txt` runs against `hook.txt` and `hook-old.txt`.
2. **LOW, design**, `internal/shellguard/rows.go:324`. Some closed heredocs are not recognised, so an edit after them still passes. Neither the row comment nor `doc.go` names these gaps, and `doc.go:22-24` claims the match reads after a closed heredoc.
   - The closing word is limited to `[A-Za-z0-9_]+`, so a delimiter such as `END-X` or `EOF.` is never closed. `sed -i` after it passes (cases `hyphen_delimiter`, `dot_delimiter`).
   - `shellLineUnitPattern` refuses `<<`, so two heredoc operators on one line (`cat <<A <<B`) never match. The edit after them passes (case `two_heredocs_on_one_line`).
3. **MEDIUM, design**, `internal/shellguard/rows.go:331`. `afterClosedHeredocScope` copies `shellScopes[0]` and appends no `shellScriptReachPattern`. So the lines after a closing line are not read the way the top level reads them.
   - `cat <<EOF\ny z\nEOF\nbash -c 'sed -i s/a/b/ f'` passes, and so does the double-quoted form, while the same `bash -c` at the top level is denied.
   - A heredoc closed inside a `bash -c '…'` script, followed by `sed -i` in that script, also passes, because no scope reads a closed heredoc inside a script.
   - Evidence: cases `heredoc_then_sh_c_edit`, `heredoc_then_dq_sh_c_edit`, `heredoc_in_sh_c_then_edit` and `top_level_sh_c_edit_baseline`.
4. **LOW, design**, `internal/shellguard/rows.go:324`. `<<-?` does not exclude a here-string (`<<<`) or an arithmetic shift. Both are read as a heredoc operator, and the lines after them as body up to the next bare-word line.
   - `cat <<<abc\nsed -i s/a/b/ f` still passes (case `herestring_then_edit_next_line`).
   - The arithmetic false denial is the last command in finding 1.

Tool calls: none. Every run cited above is already in the inherited context.
