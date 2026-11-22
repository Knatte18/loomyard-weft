```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checks I ran, one search per finding:
- **F1:** the two edited test files hold no remaining `refs/lym/` or `branch.main.` literal, so the fix covers every site.
- **F5/F4:** `seedPushCause` and `fail` have one board call site that adds the restore clause or the not-withdrawn cause (`destroy.go:1792`). The records-branch call (`:1768`) correctly passes an empty `recordsTip`.
- **F6:** the remaining bare `return err` at `pushclassified.go:200` is the `originNamespaceAt` read before any refusal, not one of the three named errors.
- **F7/F2:** no stale text remains for "at no commits", "holds its seed at" or "returns the records branch to what origin held". The only remaining `isAncestorOrEqual` caller is `resetUnnamedTip` (`pushclassified.go:396`), which is unaffected.

Tool calls:
1. Grep of the two clone test files for `refs/lym/`, `branch.main.` and `"main"`, because my context had the pre-edit text and not a search of the final state.
2. Grep of `return err` in `pushclassified.go` and of `fail(`/`seedPushCause(` in `destroy.go`, to check the error-return and call-site sets.
3. Repo-wide grep for the old message fragments and the `isAncestorOrEqual` callers, to find stale text outside the edited files.
