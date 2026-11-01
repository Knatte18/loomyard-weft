```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `sed -i`: bare, `.bak` suffix, `-Ei` cluster, flag after the script, `--in-place`, after a separator | `rows_integration_test.go` `TestRows_PatternsDenyAndAllow` rows `sed_in_place` through `sed_in_place_after_a_separator` | pinned |
| `sed` read negatives (`sed -n`, `sed 's/a/b/' f.go`, `grep -i` after a pipe, `'sed -i'` in single quotes) | the `sed_read`, `sed_to_standard_output`, `grep_flag_after_a_pipe_from_sed` and `sed_in_place_inside_single_quotes` rows, plus `allowedEverywhere` `sed_range_read` | pinned |
| `perl -pi` and `-0777 -i`; `perl -Mstrict` passes | `perl_in_place_cluster`, `perl_in_place_before_the_script`, `perl_module_switch` | pinned |
| `tee` into a source file; `tee` into `.log` passes | `tee_into_a_project_file`, `tee_into_a_log` | pinned |
| `>` and `>>` redirects (spaced, unspaced), redirect on a second line, heredoc after the redirect, redirect on the heredoc operator line | `redirect_into_a_project_file`, `append_redirect_without_a_space`, `redirect_on_a_second_line`, `heredoc_after_the_redirect`, `redirect_on_the_heredoc_line` | pinned |
| `>|` clobber redirect, named in the `redirectWritePattern` doc and in `[>|]?` | none | gap |
| Redirect negatives: `.json` capture, `/dev/null 2>&1`, redirect inside a quoted string, redirect inside a heredoc body, longer extension `.go.orig` | `output_captured_into_a_json_file`, `redirect_to_dev_null`, `redirect_inside_quotes`, `redirect_inside_a_heredoc_body`, `redirect_into_a_longer_extension` | pinned |
| A quoted redirect target (`> "f.go"`) passes; the doc comment states this | none. `redirect_inside_quotes` puts the `>` inside the quotes, not the target. | gap |
| Each `sourceFilePattern` extension (`mod`, `sum`, `yaml`, `toml`, `sh`, `ps1`, `cs`, `csproj`, `sln`, `props`, `targets`) is denied | none. Only `.go` and `.md` are exercised. | gap |
| Shell-edit pattern matches after a separator in the payload | `{"shell-edit", "x; sed -i s/a/b/ f > g.go"}` in the separator table | pinned |
| `shell-edit` row is installed in every run with no switch | `settings_test.go` `TestBuildSettings_DenyToggleMatrix` (`shellEditInstalled`, `len(preToolUse) != 4`) and `TestBuildSettings_AgentAndBashHooks` (exactly one entry) | pinned |
| Row ordering and `Prepare` entry counts (5 and 4) | `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` rows | pinned |
| Golden settings file | `settings-golden.json` with the golden comparison test. I did not look at how the golden is regenerated. | pinned, not verified |
| Python steer and `shell-edit` steer carry `editToolsRule` | `rows_test.go` `TestCheckTexts/ShippedRows` | pinned |
| The edit rule is stated in exactly one notice, so the python notice no longer repeats it | `editRuleNotices != 1` in `TestCheckTexts/ShippedRows` | pinned |
| Steer and notice contain no forbidden characters | `TestCheckTexts` and the package `init` | pinned |
| `Forms` entries `{sed,-i}`, `{perl,-i}`, `{perl,-pi}` are refused by the pattern | possibly a generic Forms-versus-pattern test. My one grep checks for it. | to verify |
| `editRulePhrases` swapped to `"never by shell"` | `parentdirective_test.go` `TestParentDirectiveScans_SyntheticRows` | pinned |
| Every parent directive embeds the new edit rule | the existing `editRulePhrases` scan over the real stencils. I did not check this. | not verified |
| `norecoverydeny_test.go` self-check row changed to `sed -n` | the row itself. It now only shows `sed -n` is not a denied recovery form, which is trivial coverage. | LOW |

**Findings**

1. LOW, class `scope`. `internal/shellguard/rows_integration_test.go`, in `TestRows_PatternsDenyAndAllow` (the shell-edit block, lines 154–178 of the diff). The `>|` branch of `redirectWritePattern` has no case. Fix: add a deny row such as `redirect_clobber_into_a_project_file`, `echo x >| f.go`.
2. LOW, class `scope`. Same block. The documented pass for a quoted redirect target, as in `echo x > "f.go"`, has no case. `redirect_inside_quotes` only covers `>` inside a quoted string. Fix: add a pass row `redirect_into_a_quoted_target`.
3. LOW, class `scope`. Same block. `sourceFilePattern`'s extension alternation is exercised only for `.go` and `.md`. A typo or drop of `yaml`, `toml`, `sh`, `cs` and the others would go unnoticed. Fix: add a table-driven deny row per remaining extension, or one row per extension class.

I could not confirm the `Forms` consistency test (see the tool-call line below), so I raise no finding on it.

**Tool calls:** none made. Everything above comes from the diff already in my context. A `Forms`-versus-pattern test would live in `rows_integration_test.go` or `rows_test.go`, outside the diff hunks I hold. I did not read it, so it is unverified.
