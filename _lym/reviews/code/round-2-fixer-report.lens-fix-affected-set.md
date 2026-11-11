```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked set: the callers of `otherFileMiss` under the `ownPackage` key, which decides the embed scope.
- `internal/testcover/testselect.go:210` passes `true` for the importing package. `OwnTestFileEmbedsInertFile` pins it.
- `internal/testcover/testselect.go:250` passes `false` for linked packages. The new `LinkedTestFileEmbedIsNoSource` row pins it.
- Those are the only two callers, so both halves of the embed scope are now pinned.
- If `embedPatterns` read linked test files, `a/a_test.go`'s `static/page.txt` directive would match the changed `a/static/page.txt`. Select would then return `MissNonGoFile`, and the row, which expects no miss, would fail.
- The row replaces `a/assets.go` with `"package a\n"`, so the committed `//go:embed static` directive cannot name `page.txt` and hide the regression.
- The cache key `embedScope{dir, withTests}` keeps the own and linked entries separate, so one cannot mask the other.

Unchecked findings: none.

Tool calls (1):
1. `grep` on `testselect.go` for `embedPatterns`, `otherFileMiss`, `withTests` and `ownPackage`. My context held only the test file, not the production call sites.
