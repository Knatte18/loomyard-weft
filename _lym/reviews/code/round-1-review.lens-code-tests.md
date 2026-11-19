```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I named the `TestRows_PatternsDenyAndAllow` row (in `internal/shellguard/rows_integration_test.go`) that would fail if it broke. Each row's expected value is a literal written in the test, not computed from the pattern code.

**Behaviour → pinning test row**
- Command position at the start of a single-quoted `-c` script → `python/behind_bash_dash_c`, `computed-rm/shell_flag_cluster`, `machine-sleep/in_a_shell_script`, `headless-claude/in_a_shell_script`, `adopt-take-hold/in_a_shell_script`, `publish/git_push_in_a_shell_script`.
- Separator inside a single-quoted script → `computed-rm/flag_words_before_dash_c` (`cd x; rm $D`).
- An escaped double-quoted string read as one unit inside a single-quoted script → `computed-rm/background_rm_in_a_shell_script_loop`. The `rm` is reachable only past `"$d/x"`.
- Command position at the start of a double-quoted script → `computed-rm/double_quoted_shell_script`, `refuse-raw/in_a_double_quoted_shell_script`.
- Separator inside a double-quoted script → `python/after_a_separator_in_a_double_quoted_shell_script`.
- A single-quoted string read as one unit inside a double-quoted script → `computed-rm/single_quoted_separator_in_a_double_quoted_shell_script`. This only shows the unit is consumed. It does not show that a `;` inside it is not a separator; see finding 4.
- `shellDashCPattern` with a flag cluster, flag words or a path prefix → `computed-rm/shell_flag_cluster`, `flag_words_before_dash_c`, `path_prefixed_shell_after_a_separator`.
- `ssh -c` gets no match → `computed-rm/ssh_dash_c`.
- `dash`, `ksh` and `zsh` through `(ba|da|k|z)?sh` → none (finding 2).
- The `do` keyword in the command-word prefix → `computed-rm/rm_after_do`, `shell-edit/sed_in_place_after_do`, `computed-rm/rm_after_do_in_a_shell_script`.
- The keywords `then`, `else`, `if`, `elif`, `while` and `until` → none (finding 1).
- A keyword as an argument word is not a command position (e.g. `echo do rm $x`) → none (finding 3).
- A command word as an argument inside a script stays allowed → `computed-rm/rm_as_an_argument_in_a_shell_script`, `python/argument_in_a_shell_script`, `headless-claude/interactive_launch_in_a_shell_script`, `publish/gh_pr_view_in_a_shell_script`.
- A shell `-c` text inside a quoted argument stays allowed → `python/shell_script_inside_a_double_quoted_argument`, `shell-edit/shell_script_inside_a_double_quoted_argument`, `computed-rm/shell_script_inside_a_double_quoted_argument`, `computed-rm/shell_script_inside_a_single_quoted_argument`.
- The Seen incident shape (a `sh -c` loop, `mktemp -d`, `rm -rf` of a variable path) → `computed-rm/background_rm_in_a_shell_script_loop`.
- `redirectWritePattern` script branch, single-quoted → `shell-edit/redirect_in_a_shell_script`. Double-quoted → `shell-edit/append_redirect_in_a_double_quoted_shell_script`. Negative with a `.json` target → `output_captured_into_a_json_file_in_a_shell_script`.
- A redirect inside a quoted string within a script is no match → none (finding 4).
- `rawGoCommandPattern` fourth alternative, single-quoted script → `raw-go/slot_variable_in_a_shell_script`. Double-quoted script → `raw-go/strand_variable_set_in_a_double_quoted_shell_script`.
- A name inside a quoted string of the script is no match (which the doc comment claims) → none (finding 4).
- `'` added as a word end in the go `all` alternative → `raw-go/all_pattern_in_a_shell_script`. The `./...` row would pass without the change, because `goWordPattern` takes the `'`.
- A package-scoped go test or a gate route inside a script stays allowed → `raw-go/package_scoped_test_in_a_shell_script`, `refuse-raw/gate_route_in_a_shell_script`.
- `topLevelReachPattern` and `commandWordPrefixPattern` extraction, which should not change top-level matching → the existing top-level rows, heredoc rows and description-field rows. The `claudeengine` settings golden pins the rendered text. Codex has no golden testdata directory.
- The `behind_bash_dash_c` flip from false to true → the row itself, with a literal expected value.

**Findings**

1. **MEDIUM**, `design`. `internal/shellguard/rows.go:37` (`commandWordPrefixPattern`): six of the seven new keywords have no test row.
   - Only `do` is pinned. A row such as `if true; then rm $x; fi` or `bash -c 'while true; do x; done'` exercises the others.
   - Removing `then`, `else`, `if`, `elif`, `while` or `until` from the alternation fails no test.
   - Evidence: grep for these keywords in `rows_integration_test.go` finds no rows; the only `do` rows are at `rows_integration_test.go:241`, `:252` and `:253`.
2. **LOW**, `design`. `internal/shellguard/rows.go:41` (`shellDashCPattern`): `dash`, `ksh` and `zsh` have no row.
   - Every new row uses `sh` or `bash`, so dropping `(ba|da|k|z)?` down to `(ba)?` fails no test.
   - The doc comment (`doc.go:11`) and the summary both name all five shells.
   - Evidence: grep for `dash|ksh|zsh` finds no rows in `rows_integration_test.go`.
3. **LOW**, `design`. `internal/shellguard/rows.go:37`: no negative row shows that a keyword used as an argument stays allowed.
   - The focus directive asks about exactly this, e.g. `echo do rm $x`.
   - The pattern accepts a keyword only right after command position. Nothing pins that, so a regression that let the prefix match anywhere would pass the suite.
   - Evidence: the new rows at `rows_integration_test.go:241–263` hold no such negative case.
4. **LOW**, `design`. `internal/shellguard/rows.go:46–51` and `:84–86`, `:112–113`: the script's quoted-string units have no negative rows. Three negative cases are missing:
   - A separator inside a quoted string of the script, e.g. `bash -c "echo 'x; rm $d'"` or `sh -c 'echo "x; rm $d"'`, both expected allowed.
   - A redirect inside a quoted string of the script, e.g. `sh -c 'echo "> f.go"'`, expected allowed.
   - A lym variable name inside a quoted string of the script, e.g. `bash -c 'grep "LYM_GATE_SLOT" f'`, expected allowed. The comment at `rows.go:99` claims this case and no row asserts it.
   - The positive rows only show that these units are consumed, not that their contents stay unread. A unit pattern that leaked into its quote would pass every row.
   - Evidence: the new rows at `rows_integration_test.go:205–209`, `:238–243` and `:251–263` are all positives or quoted-argument negatives at the top level.

Tool calls:
1. Listed `codexengine/testdata` and grepped the shellguard tests for the keywords and shell names. The manifest had not shown whether Codex keeps a golden, or whether any row names `then`, `elif`, `while`, `until`, `dash`, `ksh` or `zsh`.
