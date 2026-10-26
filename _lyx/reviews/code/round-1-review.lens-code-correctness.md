```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff for code-correctness (lens: code-correctness). The static checks I ran found no defects. I did not run the tests or the settings golden (the gate runs them).

**Per function or branch, case run, result**

- `internal/shellguard/rows.go` imports: `fmt` and `strings` are both still used (lines 130–163, 239–243). Correct.
- `commandPositionPattern`, `shellLineUnitPattern` and `shellUnitPattern`: `shellLineUnitPattern` is still used at line 65. `unquotedPrefixPattern` was its only other user. Correct.
- `rowsAfterRawGo`, with the two rows removed: `computed-rm` and `machine-sleep` are still returned. Correct.
- `TestRows_PatternsDenyAndAllow`:
  - The `allowedEverywhere` loop runs the previously tmux-tier-only allow check, plus four scripted-edit commands, against every remaining row. None of those commands matches a remaining row, as far as I can tell by reading the patterns.
  - The `in-place-edit` and `redirect-write` rows are dropped from the "payload field" table too. Correct.
- `TestBuildSettings_DenyToggleMatrix`: the PreToolUse count is 3 (`raw-go`, `computed-rm`, `machine-sleep`), which matches the two deleted rows. Correct.
- `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`: the counts drop from 6 to 4 and from 5 to 3. Consistent with the two deleted rows.
- `TestShellGuardDenies_RenderedCommandsGrepThePayload`: the `redirect-write` cases are replaced by `machine-sleep` cases. The `systemctl hibernate` deny and the `systemctl status` allow are plausible against the visible `machine-sleep` rows (`rows_integration_test.go:153–160`). The quoted-in-grep case is the one I could not confirm from the diff, so the gate run settles it.
- `editRulePhrases` (`stencils.go`):
  - The split literal `"`sed " + "-i`"` becomes `"`sed -i`"`. Because the denied-forms scan no longer lists `sed -i`, the split is no longer needed.
  - Dropping `replace_all` fits the rewritten directive, which no longer says it.
  - The directive text still contains all three remaining phrases (`sed -i`, `heredoc`, `edit or write`), so the single-statement scan still finds it.
- `norecoverydeny_test.go` self-check: the `sed -i is not denied` row expects 0 hits. This is correct once the row's `Forms` are gone. The `sed -n` and suffixed-`sed -i` rows are dropped, which is consistent.
- `settings-golden.json`: the large diff is in the area where the deny hooks for the two removed rows would be. I did not compare the bytes and never saw the diff, so a golden mismatch is only caught by the test.

**Findings**

None.

**Tool calls (2)**
1. `grep` and `sed -n` over `internal/shellguard/rows.go`, the shellguard tests and other files, plus `git diff` of the claudeengine and stencil tests. These were the callers, the leftover-reference search and the test diffs my context lacked.
2. `sed -n` and `grep` on the `rows.go` imports and the `machine-sleep` test rows, plus a `go vet` run on four packages. I needed this to confirm the imports, the surviving rows and that everything compiles. The `go vet` output came back empty.
