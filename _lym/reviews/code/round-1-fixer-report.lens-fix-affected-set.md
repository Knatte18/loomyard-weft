```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The sets I checked:
- **F1 literals:** the only non-test `"burler-review"` and `"burler"` literals left outside `internal/burlerengine/review.go` (the new constants) are pre-existing and unrelated:
  - `internal/shedadapters/burlerreview.go:26` is a log label, added in `add9f93d`, which predates this change set.
  - The `burlercli` and `configreg` literals are cobra command names and config module names, not the role name or the `.lym` directory name.
  - `internal/shedadapters/focus.go:15` is a label.
- **`reviewRolePrefix` rename:** no references to the old lowercase name remain anywhere outside `.lym` and `_lym`.
- **F7 docs:** `docs/overview.md` is the only file in `docs`, `README.md` or `contracts` that names the assist keys. Its two lines (625 and 672) now carry all eight keys.

Calls made:
1. Grepped for the `"burler-review"` and `"burler"` literals, the old `reviewRolePrefix` name, and `assist_reviewer` in the docs. My context only held the diff, not a repo-wide search result.
2. Ran `git log -S` and read `internal/shedadapters/burlerreview.go` lines 20-28. I needed to know whether the `shedadapters` literal was added by this change set, and my context didn't show that.
