```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Searches run, one per finding:
- **F1 and F4 (`sed -i`, `perl -i`, `--in-place`):** every remaining hit is intended. They are the "not denied" and "allowed" assertions in `cmd/lyx/norecoverydeny_test.go:228` and `internal/shellguard/rows_integration_test.go:209`, the `refscanner_test.go` case, the new directive wording in `contracts/stencils/edit/edit-directive.md:6` and its `doc.go` twins, the `editRulePhrases` list in `contracts/stencils/stencils.go:568`, and the dated research notes. No other background, doc or deny-guardrail list still names a removed deny. The `COMMANDS.md` hit says a script was not edited with `sed -i`, which is unrelated.
- **F2 (`unquotedPrefixPattern`):** no references remain outside `_lyx/` and `.lyx/`.
- **F3 (edit-directive prose assertions):** the only assertion on the directive's static prose is the single "Edit or Write" check in `internal/editdirective/editdirective_test.go`. I did not reread the test.
- **Removed rows by name (`redirect-write`, in-place):** the `docs`, `pattern`, `PATTERN.md`, `shellguard/doc.go` and `claudeengine` sweep found nothing stale. The `docs/overview.md:267` hit is the unrelated "redraw-in-place" pane loop.

Findings left unchecked: none.

Tool calls:
1. Grep for `sed -i`, `perl -i` and `--in-place` across md, go and yaml, to find sites beyond the four the review named.
2. Grep for `unquotedPrefixPattern`, to confirm no reference to the deleted symbol remains.
3. Grep for in-place, redirect-write and scripted-edit mentions in docs and pattern files, to find stale mentions of the removed rows.
4. A `sed -n` read of two sites, refused by the hook, so I did not retry it.
5. Read `internal/shellguard/rows_integration_test.go:195-215`, to see whether the `sed -i` hit there was an allow list.
6. Read `contracts/stencils/stencils.go:558-571`, to see whether the `sed -i` hit there was a stale deny.
