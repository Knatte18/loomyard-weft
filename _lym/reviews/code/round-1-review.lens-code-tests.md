```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it.

**Per behaviour, the test that pins it**

1. **Compare path of `TestBuildSettings_MatchesGolden`** (`settings_test.go:1186-1212`): the golden lacks a label, the settings differ, the notice differs, or the golden holds a label the test does not render.
   - Pinned by the test itself against `testdata/settings-golden.json`. Each of the four checks fails when it should.
   - The settings and notice come from `buildSettings` and `buildDenyNotice` with the same arguments as at the base. The only change is that the render happens before the read.
   - Correct.
2. **`-update` mode** (`settings_test.go:1153`, `1178-1185`): rewrites the golden with `MarshalIndent` at two-space indent plus a trailing newline, and compares nothing.
   - No test pins it.
   - The compare path pins only the file that was committed, so it would catch a rewrite that differs from that file only after someone has run `-update` and committed the result.
   - This is test tooling, not product behaviour, so a test of the test is not warranted. No finding.
3. **Session-bus row `system_flag_of_a_later_dbus_send`** (`rows_integration_test.go:316`, expect `true`, meaning denied).
   - Traced `dbus-send / x.y; dbus-send --system / a.b` through `sessionBusCommandPattern` (`rows.go:424`):
     - `/` and `x.y` each match `unquotedWordOtherThan("--system")`, since each opening unit leaves out only `-`.
     - `x.y` stops at `;`, because the unit leaves it out.
     - `[[:space:]]*` matches nothing, and `searchEnd` (`rows.go:420`) matches the `;`.
     - So the first dbus-send matches, the row denies, and `true` is the right direction.
   - It pins the later-command property it names: an exception written as a separate must-not-match pattern would let this payload through.
   - Overlap with `system_flag_of_a_later_command` (`rows_integration_test.go:315`, `dbus-send / x.y &amp;&amp; ls --system`, true):
     - Both pin "a later command's `--system` does not release the earlier dbus-send".
     - The old row reaches the end of the scan through `&amp;`. The new row is the only session-bus row that reaches it through `;`, so dropping `;` from `searchEnd` fails only the new row.
     - It is not wholly redundant. No finding.
4. **Doc-comment changes** in `internal/shellguard/doc.go:30-41` and `rows.go:37-38,428-431`: static prose, exempt.

**Findings**

None.

**Tool calls**

None. My context already held the test table, the pattern code and the test function.
