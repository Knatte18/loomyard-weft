```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F3 cover every member of the set each premise is about.

Left unchecked: none.

1. **LOW**: `internal/gatecli/scopes_integration_test.go:66`, the `scopesHub.plan` field comment, is a twin of F2 that was left unchanged.
   - It still reads "written under the pair's never-tracked plan directory".
   - That directory is `<pair>/_lym/plan` (`planparser.PlanDir(f.pair)`, line 120). F2's premise is that `_lym` is the tracked records namespace, which the code repository only excludes, so "never-tracked" is the wrong reason there.
   - The fixer corrected the sentence in `_lym/landing/summary.md` to "which the pair's code repository excludes", but this change set's own comment keeps the old claim. Something like "under the pair's plan directory, which its code repository excludes" would agree with it.

No other findings:
- **F1:** the grep showed other rows that run `--all` before any commit. The only derivation-base assertions I know of are in the three adjacent rows at lines 1093–1149. The first and third already use a green commit distinct from the merge base, and the second is the one fixed. I judged the other `--all`-first rows by their names only: they assert carry or run behaviour, which does not need the green commit to differ from the merge base. I did not re-read their bodies.
- **F3:** the search found no other "every row builds its own hub" claim about `scenarioHub`. Summary lines 15–16 already exclude the standalone and outside-hub rows.

Tool calls:
- An `awk` over `TestGateTest_Scopes` for rows running `--all` before any commit (F1's set). My context held only parts of that file.
- A `grep` over the gatecli integration tests and the summary for "never-tracked" and "every/each row" wording (the F2 and F3 sets). My context had no whole-set search for those phrases.
