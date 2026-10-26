```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. Cases are listed per function or branch, then the findings.

**Per function or branch (case run, verdict)**
- `rows.go` deletions, `Rows` and `rowsAfterRawGo`: I grepped for every deleted symbol (`unquotedPrefixPattern`, `notPrefixPattern`, `projectTargetPattern`, `redirectTargetEndChars`, `inPlaceCommandWordsPattern`, both steers and notices, both pattern vars). Only one dangling mention is left, a comment (see F1), so the package compiles.
- `rows.go` surviving helpers: `commandFieldPattern`, `shellUnitPattern`, `shellLineUnitPattern`, `doubleQuotedOpenPattern` and `commandPositionPattern` are still used by the python, raw-go and computed-rm patterns at lines 65, 119 and 120. Nothing the removal left unused remains, and nothing that is still needed was deleted. Correct.
- Row order in `rowsAfterRawGo`: the remaining entry (computed-rm) is unchanged and no other row moved. Correct.
- `hookcmd_integration_test.go` `TestShellGuardDenies_RenderedCommandsGrepThePayload`, case `systemctl hibernate`: the command rendered for the machine-sleep row should deny, and the test expects `denyJSON(row.Steer)`. The expected outcome is consistent, but I did not open the machine-sleep pattern, so I did not run the quoted-argument case `grep 'x; systemctl hibernate' f` against it. Not confirmed.
- Same test, row `computed-rm`: unchanged. Correct.
- `claudeengine/settings.go` comment: it matches the remaining rows. Correct.
- `edit-directive.md`: the text is marker-free (no `{{`), as the `editdirective` test requires. Correct.
- `stencils.go` `editRulePhrases`: the phrase `"`sed -i`"` is now a plain literal, and the old split that kept the no-denied-recovery scan from reading it is gone. That is correct only because the scan no longer treats `sed -i` as denied. `norecoverydeny_test.go` pins this with the row `{"sed -i is not denied", ..., 0}`. Correct.
- `editRulePhrases` drops `replace_all`, and the directive no longer contains it. Consistent.

**Findings**

F1 — LOW, consistency
- Location: `internal/shellguard/rows.go:61`
- Summary: the `commandPositionPattern` comment still says the separator is reached "through a run of shell units as unquotedPrefixPattern reads them". That constant was deleted in this diff, so the comment now names a symbol that does not exist.
- Evidence: the grep output above shows `unquotedPrefixPattern` only at this comment line.
- Fix: reword it to "through a run of shell units" with no reference to the deleted constant.

F2 — LOW, consistency
- Location: `internal/shuttleengine/claudeengine/hookcmd_integration_test.go:227-228`
- Summary: the doc comment says "A row whose pattern holds a single quote and a row of several patterns", which described `redirect-write` and the multi-pattern rows. After the row swap it is unclear that the machine-sleep and computed-rm rows still cover a quote-bearing pattern.
- Why it matters: the test is meant to pin sh-quoting of a pattern containing `'`. Every shell-unit pattern embeds `'[^']*'`, so it probably still does. Please confirm, or reword the comment.
- Evidence: `rows.go:55-57`, `hookcmd_integration_test.go:227-228`.
- Fix: verify that the machine-sleep row (or computed-rm) pattern contains a literal `'`, then keep or reword the comment.

No `design` findings.

Tool calls (1 of 6):
1. `grep` for the deleted symbols and a `sed -n 215,260p` of `hookcmd_integration_test.go`, because my context lacked the other uses of the shared helper constants and the full test body.
