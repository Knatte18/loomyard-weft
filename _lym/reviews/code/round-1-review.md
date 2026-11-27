---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go#closedHeredocPattern
    summary: The match does not skip a heredoc body that holds a line with a bare word alone. A later body line, or quoted text after such a line, that opens with sed -i, perl -i or tee into a source file is now denied though it edits nothing, against the entry's 'while still skipping the heredoc body'.
    fix: "In internal/shellguard/rows.go, tie the closing line to the operator's word for a closed set of delimiter words. Declare one named constant list, such as EOF, END, EOT and PY. For each listed word render an alternative: the operator `<<-?` with the word bare, single-quoted or JSON-escaped double-quoted, the rest of the operator line, then body lines that are not that word alone after optional tabs, then that word's closing line. Build the not-that-word line the way unquotedWordOtherThan builds a word that is not its argument. Keep the generic bare-word closing only for an operator word that is none of the listed words, so a listed operator never also matches the generic form. Then fix the row comment at rows.go:837-838 and the doc.go:22-26 sentences to state the remaining false denial for unlisted words only. In internal/shellguard/rows_integration_test.go add allow rows with the body skipped: `cat > .scratch/msg.txt <<'EOF'\\nSummary\\n\\nsed -i is now refused\\nEOF\\ngit commit -F .scratch/msg.txt` and `cat > .scratch/notes.txt <<EOF\\nFindings\\nperl -pi slipped past\\nEOF`. Add a denied row pinning the documented false denial for an unlisted delimiter word. If the fixer judges the tied match unworkable, it leaves the code and defers to the run's parent, who records or refuses the trade-off with lym loom decision add."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go#afterClosedHeredocScope
    summary: After a closed heredoc a shell -c script is not read. cat <<EOF / y z / EOF / bash -c 'sed -i s/a/b/ f' passes, single- or double-quoted, while bash -c 'sed -i s/a/b/ f' alone is denied. A heredoc closed inside a shell -c script hides the edits after it. doc.go:18-19 claims a guard refuses inside the script what it refuses outside it, and neither doc.go:22-26 nor the row comment names the exception.
    fix: "In internal/shellguard/rows.go, render the after-heredoc alternative for every scope, not shellScopes[0] alone. Factor the after-heredoc prefix, commandFieldPattern + `(` + shellUnitPattern + `*` + closedHeredocPattern + `)+` + topLevelReachPattern, into one named constant. Build two more scopes from shellScopes[1] and shellScopes[2]: each position is that prefix + singleQuotedScriptReachPattern or doubleQuotedScriptReachPattern + commandWordPrefixPattern. Add `|(` + position + inPlaceEditCommandIn(scope) + `)` for each to inPlaceEditCommandPattern. In rows_integration_test.go add denied rows `cat <<EOF\\ny z\\nEOF\\nbash -c 'sed -i s/a/b/ f'` and `cat <<EOF\\ny z\\nEOF\\nbash -c \\\"sed -i s/a/b/ f\\\"`. For a heredoc closed inside a shell -c script, which the scope change does not reach, name it as a pass in the row comment at rows.go:835 and in doc.go:22-26."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go#closedHeredocPattern
    summary: Some closed heredocs are never recognised, so an edit after them passes, and no comment names these gaps. A delimiter word holding - or . is one, such as END-X or EOF.; its closing word is limited to [A-Za-z0-9_]+. Two heredoc operators on one line are another, such as cat <<A <<B; shellLineUnitPattern stops at the second <<. doc.go:23 says the match reads the lines after a closed heredoc's closing line without that limit.
    fix: "In internal/shellguard/rows.go#closedHeredocPattern, admit `<<-?` followed by its word as a unit of the operator line's rest, so a second operator on the line is read. Repeat the body-and-closing part once per operator. Widen the closing word to `[A-Za-z0-9_.-]+`. Whatever stays unread, name it as a pass in the shell-edit row comment at rows.go:835 and in doc.go:22-26. Add denied rows `cat <<'END-X'\\ns/a/b/;\\nEND-X\\nsed -i s/a/b/ f` and `cat <<A <<B\\na a\\nA\\nb b\\nB\\nsed -i s/a/b/ f`."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shellguard/rows_integration_test.go:194
    summary: No row pins a closed heredoc with an empty body. Every new deny row has a body line, and rows 194 and 197 have the bare-word body line x, which can itself act as the closing line, so changing the body repeat from * to + would pass the whole table.
    fix: "In internal/shellguard/rows_integration_test.go, after the row at line 194, add the row {\"shell-edit\", \"sed_in_place_after_a_closed_heredoc_with_an_empty_body\", \"cat > .scratch/a <<EOF\\nEOF\\nsed -i s/a/b/ f\", true}."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shellguard/doc.go:15
    summary: The package doc still says a command-shaped row reads at most one heredoc operator plus its line. The shell-edit in-place match now reads past one or more closed heredocs and the lines after them, so the general statement contradicts the exception at doc.go:22-23.
    fix: "In internal/shellguard/doc.go, end the line-15 sentence's list with the shell-edit row's in-place edit match as the exception that reads past closed heredocs. Or move that exception next to line 15 and drop it from line 22, so the package doc states the reach once."
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: The Merge danger section names shellguard as the blast radius. The change alters what every lym session's Bash guard refuses on Claude and Codex, through the rendered settings and Codex's guards. The Design decisions bullet calls the false denial one of an in-place edit, but the denied text also includes prose that only names the command.
    fix: "In _lym/landing/summary.md, set the Blast radius to the shell-edit guard of every lym session on Claude and Codex. Reword the Design decisions bullet so it says that any text opening a later body line with an in-place edit spelling is denied, prose naming the command included. Bring both in line with what F1's fix leaves."
---

### [MEDIUM] The heredoc body is not skipped when it holds a bare-word line

**Location:** `internal/shellguard/rows.go#closedHeredocPattern`

**Issue:** grep -E has no backreference, so the pattern takes any line holding a bare word alone as the closing line.
A body line such as `Summary`, `Findings` or `done` therefore ends the skipped body.
Each later body line is then read in command position.
`heredocBodyLinePattern` also crosses a JSON-escaped `\"`, so the "body" can run into a double-quoted string after the real closing line.
The rendered hook from the golden denies these commands at HEAD and allows them at base `4aa85ab3a`:

- `cat > .scratch/msg.txt <<'EOF'\nSummary\n\nsed -i is now refused after a heredoc\nEOF\ngit commit -F .scratch/msg.txt`
- `cat > .scratch/notes.txt <<EOF\nFindings\nperl -pi slipped past the guard\nEOF`
- `cat > .scratch/fix <<'EOF'\nfor f in a b; do\n  echo "$f"\ndone\nsed -i s/c/d/ g\nEOF\nls`
- `x=$((1<<2))\necho "a\nb\nsed -i x"`, where an arithmetic shift is read as a heredoc operator

The entry asks to read the lines after the closing delimiter "while still skipping the heredoc body".
The row comment (`rows.go:837-838`) and the summary record the false denial as a design decision.
The record has no `Added after Discussion` entry for it, and a tied match for the usual delimiter words keeps the body skipped.
An agent writing a commit message or notes about this very guard hits the denial.
No test row shows the documented limit (the code-tests lens's second finding, folded in here).

### [MEDIUM] Lines after a closed heredoc are not read through a shell -c script

**Location:** `internal/shellguard/rows.go#afterClosedHeredocScope`

**Issue:** The scope copies `shellScopes[0]`, and its position has no `shellScriptReachPattern` step.
`cat <<EOF\ny z\nEOF\nbash -c 'sed -i s/a/b/ f'` and its double-quoted form pass.
`bash -c 'sed -i s/a/b/ f'` on its own is denied (cases `heredoc_then_sh_c_edit`, `heredoc_then_dq_sh_c_edit`, `top_level_sh_c_edit_baseline`).
`bash -c 'cat <<EOF\ny z\nEOF\nsed -i s/a/b/ f'` passes too, since no scope reads a closed heredoc inside a script.
`doc.go:18-19` says a guard refuses inside a shell -c script what it refuses outside it.
The new reach breaks that claim, and `doc.go:22-26` and the row comment state no exception.

### [LOW] Delimiters holding - or . and two operators on one line are never read past

**Location:** `internal/shellguard/rows.go#closedHeredocPattern`

**Issue:** `cat <<'END-X'\ns/a/b/;\nEND-X\nsed -i s/a/b/ f`, `cat <<EOF.\ns/a/b/;\nEOF.\nsed -i s/a/b/ f` and `cat <<A <<B\na a\nA\nb b\nB\nsed -i s/a/b/ f` all pass at HEAD.
The closing word class `[A-Za-z0-9_]+` and the operator line's `shellLineUnitPattern`, which stops at a second `<<`, account for them.
Neither the row comment nor `doc.go` names these passes, and `doc.go:23` states the reach without them.

### [LOW] No row pins a closed heredoc with an empty body

**Location:** `internal/shellguard/rows_integration_test.go:194`

**Issue:** `closedHeredocPattern` lets the body repeat zero times.
Every new deny row has a body line, and a bare-word body line such as `x` can itself serve as the closing line.
A mutation requiring one body line would survive the table.
The rendered hook denies `cat > a <<EOF\nEOF\nsed -i s/a/b/ f` today, so the row pins current behaviour.

### [LOW] doc.go line 15 still limits every row to one heredoc operator's line

**Location:** `internal/shellguard/doc.go:15`

**Issue:** The opening reading rule says a command-shaped row reads "at most one heredoc operator plus its line".
Lines 22-23 then except the shell-edit row's in-place match, so the doc states the reach twice and the first statement is no longer true for every row.

### [LOW] The change description understates the blast radius and the false denial

**Location:** `_lym/landing/summary.md`

**Issue:** `**Blast radius:** shellguard` names the package.
What the diff can break is the Bash guard of every lym session on both engines: the Claude settings golden carries the new pattern, and Codex renders `row.Patterns` generically.
The Design decisions bullet says "an in-place edit on a later body line after such a line is falsely denied".
The denials found here are of prose naming the command, which edits nothing.

## Rejected

- code-correctness finding 4, here-string and arithmetic `<<` read as a heredoc operator: `cat <<<abc\nsed -i s/a/b/ f` also passes at base, through `topLevelReachPattern`, so the pass predates this diff and the entry asks nothing of it. The arithmetic false denial is folded into F1.
- code-scope finding 1 is the same defect as F1 and is folded into it, filed `design` since the entry's clause is the correctness bar for the pattern.
- code-scope finding 2 duplicates F2 (shell -c) and F3 (delimiter classes, two operators on one line).
- code-affected-set finding 2 duplicates F2's doc claim and is folded into it.
- code-tests finding 2, no row pinning the documented false denial, is folded into F1's fix. That fix replaces the limit with allow rows for listed delimiters and pins the remainder.

## Notes on the focus entries

- Cost and size: the shell-edit hook grew from 4592 to 5603 bytes in `settings-golden.json`, far below any argument-length limit. Under GNU grep a 110 KB command with five 400-line heredocs took 32 ms with no match and 64 ms with an edit at its end. No finding.
- The `Root-Cause:` trailer on 91c8b0bda names the defect the fix removes: `topLevelReachPattern` reads only a heredoc operator's own line. The entry's own command shape is denied at HEAD and passed at base.
- The allow rows `sed_in_place_inside_a_heredoc_body` and `..._before_a_later_command` would fail if the body were read as command lines. They have no bare-word line before the edit, which is why they miss F1.
- Commands a closed heredoc wraps in `$( … )` or `( … )`, a blank, indented or comment line before the edit, `cd y &&`, `do` and `| xargs` after the closing line are all denied correctly.

## Focus departures

- "Run the changed shell-edit pattern as the rendered hook runs it (grep -E over the JSON payload) ... check its size in internal/shuttle claudeengine testdata/settings-golden.json": I did not read the 65 testdata diff pieces whole. The golden is the renderer's output, which `TestBuildSettings_MatchesGolden -update` writes. I extracted the rendered shell-edit hook command with jq at HEAD and at the base, and ran and measured those instead.
