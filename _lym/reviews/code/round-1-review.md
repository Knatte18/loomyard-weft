---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shellguard/rows.go#shellScriptReachPattern
    summary: Argument-word scans that follow a command word reached inside a shell -c script run past the script's closing quote into later commands, adding false denies the base did not have.
    fix: "Bound every argument scan that follows a script-reached command word to the script. Give the script branch its own argument-word unit, the single-quoted script's argument word excluding ' and the double-quoted script's excluding the JSON-escaped double quote, and use it for the sed, perl, tee, claude, lym adopt, gh/git publish and go argument runs when command position was reached through shellScriptReachPattern. The simplest route is to render each command-shaped row as two alternatives, the top-level one unchanged and a script one built from shellScriptReachPattern plus script-bounded argument patterns. Add allowed rows to TestRows_PatternsDenyAndAllow in internal/shellguard/rows_integration_test.go: headless-claude `bash -c 'claude --version' && echo 'run -p'`, shell-edit `bash -c 'sed -n 1p x' ; bash -c 'grep -i foo y'`, adopt-take-hold `bash -c 'lym adopt b' && echo 'x --take-hold'` and raw-go `sh -c 'go build ./cmd/lym' all`. Regenerate internal/shuttleengine/claudeengine/testdata/settings-golden.json."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/shellguard/rows.go#commandWordPrefixPattern
    summary: The shell keywords do, then, else, if, elif, while and until widen every command-shaped row at the top level too, a change the Goal does not license and no Added-after-Discussion entry records.
    fix: Leave the code in place and defer this finding to the run's parent, who records the top-level widening with `lym loom decision add` or refuses it; recording or refusing it lies outside the fixer's write surface. Only if the parent refuses it, restrict the keywords to positions reached through shellScriptReachPattern and drop the rm_after_do and sed_in_place_after_do rows.
    origin: lens:code-scope
  - id: F3
    severity: MEDIUM
    class: consistency
    location: internal/shuttleengine/claudeengine/doc.go:706
    summary: The claudeengine package doc's per-row guardrail lists still name bash -c as a way past five rows and say a sed -i after do or then passes, which the diff now refuses; the change description's claim that the docs no longer list bash -c is therefore untrue.
    fix: In internal/shuttleengine/claudeengine/doc.go, change `bash -c` to a nested `bash -c` or drop it in favour of a script file in the raw go row's list (line 706), the refuse-raw row's (723), the shell-edit row's (733), the computed-rm row's (747) and the headless claude row's (761), and drop the `sed -i` after `do` or `then` clause at line 734, matching the rows.go row comments.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: design
    location: internal/shellguard/rows.go#rawGoCommandPattern
    summary: Adding ' to the word-end class of the go test/build/vet alternative newly denies a top-level command such as go test -run 'TestA all' ./internal/x, which the base allowed.
    fix: Keep the base word-end class for the top-level go alternative and admit ' as a word end only in a script-reached alternative (the same split F1 makes), so `bash -c 'go test all'` stays denied and top-level `go test -run 'TestA all' ./internal/x` stays allowed; add that top-level command as an allowed raw-go row.
    origin: handler
  - id: F5
    severity: LOW
    class: design
    location: internal/shellguard/rows.go#rawGoCommandPattern
    summary: The script is not read the same way as the top level in two places, so two existing refusals stop at the script; a lym variable expanded inside a double-quoted string of a single-quoted script, as in sh -c 'echo "$LYM_GATE_SLOT"', and a heredoc operator's line inside a script, as in sh -c 'cat <<EOF | rm $d', both pass.
    fix: Add a raw-go alternative mirroring the top-level double-quoted-expansion one inside a single-quoted script (script reach, then singleQuotedScriptUnitPattern units, then an escaped double quote opening a string holding $ or ${ and the name), and give shellScriptReachPattern and the script branch of redirectWritePattern a heredoc-operator-line branch like topLevelReachPattern's. Add denied rows for both commands above. If either is left out, say so in the commandPositionPattern doc comment, which now claims the script is read the same way.
    origin: assist:assist
  - id: F6
    severity: LOW
    class: scope
    location: internal/shellguard/rows_integration_test.go#TestRows_PatternsDenyAndAllow
    summary: Behaviour the diff adds has no row; only do of the seven keywords is pinned, no row uses dash, ksh or zsh, and no negative row pins a keyword as an argument word or the quoted strings inside a script staying unread.
    fix: "Add rows to TestRows_PatternsDenyAndAllow: denied computed-rm `if true; then rm $x; fi` and `bash -c 'while true; do rm $x; done'`; denied computed-rm `zsh -c 'rm $d'`; allowed computed-rm `echo do rm $x`; allowed computed-rm `sh -c 'echo \"x; rm $d\"'` and `bash -c \"echo 'x; rm $d'\"`; allowed shell-edit `sh -c 'echo \"> f.go\"'`; allowed raw-go `bash -c 'grep \"LYM_GATE_SLOT\" f'`."
    origin: lens:code-tests
---

### [MEDIUM] Argument scans run past the end of a shell -c script

**Location:** internal/shellguard/rows.go#shellScriptReachPattern
**Issue:** `shellScriptReachPattern` stops at the script's closing quote, but the rows' argument-word patterns that follow the command word do not.
`shellArgumentPattern` (and `goArgumentsPattern` for the go row) reads the script's closing `'` as the opening of a new quoted word, so the scan crosses `&&` and `;` into later commands.
Probing the rendered hooks from the base and HEAD goldens:
`bash -c 'claude --version' && echo 'run -p'` is denied by headless-claude,
`bash -c 'sed -n 1p x' ; bash -c 'grep -i foo y'` by shell-edit,
`bash -c 'lym adopt b' && echo 'x --take-hold'` by adopt-take-hold,
and `sh -c 'go build ./cmd/lym' all` by raw-go.
The base allowed all four, and no row comment names these false denies.

### [MEDIUM] The keywords widen the top level beyond the Goal

**Location:** internal/shellguard/rows.go#commandWordPrefixPattern
**Issue:** The Goal asks for the script of a shell -c to be read like a top-level command; it does not ask for the top level to change.
`commandWordPrefixPattern` is shared, so `for f in a b; do sed -i s/a/b/ $f; done` is now denied at the top level (probed: allowed at base, denied at HEAD), as is `for f in a b; do rm $f; done`.
The change description's "Design decisions" names the widening as going past the record's idea, and its reason is sound: a loop body is the incident's natural shape, and reading the script like the top level argues for one keyword rule in both places.
It is still unlicensed: the record holds no Added-after-Discussion entry and no commit carries a trailer for it, so the parent decides.

### [MEDIUM] claudeengine's package doc still lists bash -c as a bypass

**Location:** internal/shuttleengine/claudeengine/doc.go:706
**Issue:** The diff updates lines 681 and 687 but leaves the per-row guardrail lists at 706, 723, 733, 747 and 761 naming `bash -c` as a way past the raw go, refuse-raw, shell-edit, computed-rm and headless claude rows, and line 734 saying a `sed -i` after `do` or `then` passes.
The diff's own rows refuse each of these.
`_lym/landing/summary.md` claims the package docs "no longer list `bash -c` as a way past the guard", which this leaves untrue.

### [LOW] The quote word-end newly denies a top-level -run filter

**Location:** internal/shellguard/rows.go#rawGoCommandPattern
**Issue:** The `'` added to the word-end class after `all` applies at the top level as well as in a script.
`go test -run 'TestA all' ./internal/x` and `go vet -run 'x all' ./internal/x` were allowed at base and are denied at HEAD.
The row comment already admits a false deny inside a quoted `-run` argument, so this is documented, but the diff widens it at the top level when only the script needed it.

### [LOW] Two top-level refusals do not reach into the script

**Location:** internal/shellguard/rows.go#rawGoCommandPattern
**Issue:** `echo "$LYM_GATE_SLOT"` is denied at the top level, but `sh -c 'echo "$LYM_GATE_SLOT"'` passes: the script branch reads the inner double-quoted string as one unit and has no expansion alternative.
The row comment says only names outside a quoted string of the script match, so the gap is stated, yet the Goal asks for the same refusals.
Likewise `topLevelReachPattern` reads a heredoc operator's line, but the script branches stop at `<<`, so `sh -c 'cat <<EOF | rm $d'` and `sh -c 'cat <<EOF > f.go'` pass where their top-level forms are denied.
The `commandPositionPattern` comment says the script is read "the same way".

### [LOW] New behaviour without rows

**Location:** internal/shellguard/rows_integration_test.go#TestRows_PatternsDenyAndAllow
**Issue:** Removing `then`, `else`, `if`, `elif`, `while` or `until` from the keyword alternation, or narrowing `(ba|da|k|z)?sh` to `(ba)?sh`, fails no row.
No negative row pins that a keyword used as an argument stays allowed, or that a separator, redirect or lym variable name inside a quoted string of a script stays unread; the positive rows show only that those units are consumed.

## Rejected

- Assistant F1, escaped double quotes inside a double-quoted script: the miss of `bash -c "echo \"hello\"; rm \$d"` is the JSON-escaped-quote misread the `commandPositionPattern` comment already documents, and the false deny of `bash -c "echo \"x; rm \$d\""` is denied at base too, so the diff introduces neither.
- Assistant F4, an outer argument after the script read as a go argument: same root cause as F1 and folded into it.
- Lens code-affected-set finding 2, the change-description claim: same defect as F3 and folded into it.
- Lens code-scope S2: folded into F5.
- Lens code-tests findings 1 to 4: folded into F6 as one test-coverage finding; the keyword gap is LOW, since one alternation is pinned by its `do` rows and the missing rows are coverage, not a wrong result.

## Review evidence

The range is `git diff 7346d6a8..HEAD`, four commits; no trailers.
The rendered hooks of the first entry of `settings-golden.json` at base and HEAD were run with `sh` over `jq`-built PreToolUse payloads.
The Seen command, `sh -c 'd=$(mktemp -d); for i in 1 2; do mkdir "$d/x" & rm -rf "$d/x"; done'`, is denied by computed-rm at HEAD, so the Goal's core case lands.
`echo do rm $x`, `git commit -m "x" && echo if rm $x`, `ssh -c`, `fish -c` and an fd duplication inside a script stay allowed.
`topLevelReachPattern` is the base reach group unchanged, heredoc branch included.
The flip of `behind_bash_dash_c` is the behaviour the Goal asks for, not a loosened check.
