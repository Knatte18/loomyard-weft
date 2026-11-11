```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task: check that the added `LinkedTestFileEmbedIsNoSource` row is correct and fails with the defect in place.

**Walk-through**
- The row overrides `a/assets.go` to `package a\n`. It overrides `a/a_test.go` to a file with `//go:embed static/page.txt`. Both go into the committed set. The map is built from `committed`, so the recorded hashes for `a/assets.go` and `a/a_test.go` equal their working copies. Only `a/static/page.txt` (`hello` to `changed`) is then a changed file.
- `linkedChanges` reaches the `!isGoSourceFile` branch and calls `otherFileMiss("a", "static/page.txt", false)`. The inert predicate accepts `.txt`, so it goes on to `embedPatterns("a", false)`.
- `ownedFiles("a")` lists `a/a_test.go`, because it is in `WorkingFiles`, but `embedPatterns` skips it when `withTests` is false. `a/assets.go` has no directive and `a/a.go` has none either. The result is no patterns, so no miss. No function changed, so `Tests` is nil.
- The row has no `wantMiss` and no `want`. That matches the existing no-want rows, so a nil `Tests` compares equal.

**Fails with the defect in place**
- If `linkedChanges` or `otherFileMiss` passed `withTests` true, `embedPatterns` would parse `a/a_test.go` and collect `static/page.txt`. `embedded` would then match, giving `MissNonGoFile`. The row expects no miss, so it fails.
- `a/assets.go` is overridden, so its `//go:embed static` can't produce a miss that would mask the result. The new files parse, so there is no spurious `MissParse`.
- The `embed` import is in the fixture's fake `GoRoot`. Linked packages only go through `importAllowed` for added imports in changed Go files, and none are changed here.

**Callers:** No signatures or production code changed. The change is test-only: one table row, and `go test` passed for `./internal/testcover`.

No further tool calls were needed beyond the two below.

Tool calls made:
1. `sed`/`cat` over `internal/testcover/testselect.go` (`linkedChanges`, `otherFileMiss`, `embedPatterns`) and `testselect_test.go` (fixtures). I used it to trace the branch and the fixture's recorded hashes; my context held only the test table span.
2. `grep` for `ownedFiles`. I used it to confirm `a/a_test.go` is listed among the package's owned files, so the row would catch a `withTests` regression.
