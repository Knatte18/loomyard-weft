```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the row that pins it** (`internal/shellguard/rows_integration_test.go`):
1. `sed -i` on the line after a closed heredoc is refused: `sed_in_place_after_a_closed_heredoc` (:194).
2. The entry's own shape is refused (`perl -pi` cluster after a quoted-delimiter heredoc, after `;`): `perl_in_place_cluster_after_a_closed_heredoc` (:195).
3. A `<<-` heredoc closed by a tab-indented line: `sed_in_place_after_a_tab_indented_closing_line` (:196). It fails if `(\\t)*` is dropped.
4. A separator on a later line reaches command position (the `topLevelReachPattern` suffix in `afterClosedHeredocScope`): `sed_in_place_after_a_separator_after_a_closed_heredoc` (:197).
5. The body is skipped: `sed_in_place_inside_a_heredoc_body` (:198) and `..._before_a_later_command` (:199). Both would deny if body lines were read as command lines, so they fail if skipping breaks.
6. A heredoc with no closing line is not read past: `sed_in_place_inside_a_heredoc_without_a_closing_line` (:200).
7. A non-edit `sed` after a closed heredoc passes: `sed_read_after_a_closed_heredoc` (:201).
8. The settings render of the changed pattern: `TestBuildSettings_MatchesGolden` (`internal/shuttleengine/claudeengine/settings_test.go:1170`) against `testdata/settings-golden.json`.
9. The `inPlaceEditCommandIn` alternatives in the three existing scopes are unchanged and still pinned by the existing rows (:158–:165, :171, :205, :208).
10. A closed heredoc with an empty body (`(heredocBodyLinePattern)*` taking zero lines): none.
11. The documented false denial (a bare-word body line taken as the closing line, so a later body line is read as a command): none.
12. `tee` into a source file after a closed heredoc: no row of its own. Its alternative is shared with the top level (pinned by `tee_into_a_project_file`, :171), and the new scope's position is pinned by rows 1–4, so I raise no finding.

**Findings**

- **LOW, design** at `internal/shellguard/rows_integration_test.go:194`: no row pins a closed heredoc with an empty body.
  - Evidence: `internal/shellguard/rows.go:324` lets the body repeat zero times.
  - Every new deny row has at least one body line, and rows 194 and 197 have a bare-word body line (`x`) that can itself act as the closing line. A mutation that requires at least one body line (`)*` to `)+`) would survive every row.
  - Fix: add a row `cat > .scratch/a <<EOF\nEOF\nsed -i s/a/b/ f`, denied.
- **LOW, design** at `internal/shellguard/rows_integration_test.go:198`: no row pins the false denial the diff adds and documents.
  - Evidence: the row comment at `internal/shellguard/rows.go:837-838` says an in-place edit spelling after a bare-word body line is falsely denied.
  - Every allow row's body has no bare-word line before the edit, so the table neither shows nor pins this documented limit.
  - My run of the rendered hook denies `cat > .scratch/notes.txt <<EOF\nFindings\nperl -pi slipped past the guard\nEOF`.
  - Fix: add a denied row naming the limit, such as `sed_in_place_after_a_bare_word_line_inside_a_heredoc_body_is_falsely_denied`.

Tool calls: none. The inherited context already held the test table, the pattern and the hook runs.
