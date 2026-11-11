```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, covering F1 (every site stating how skips count against `--max`) and F6 (every failure path).

**No findings.**

- **F1 (every site that states how skips count against `--max`):**
  - `internal/gatecli/testmap.go:79-81`, `testmap.go:360`, `doc.go:991` and `doc.go:1004` agree with the new behaviour, and `doc.go:1023` correctly keeps the after-tests skip counting.
  - `docs/overview.md:922` says only that such a package is skipped and says nothing about `--max`, so it stays true.
  - The only caller of `mapPackage` is `testmap.go:406`. Every return in `testmappackage.go` carries the new two-value form, and only the before-build dirty return (line 94) yields `true`.
- **F6 (failure paths):**
  - **Covered by a row:** the `go list` failure (line 80), the spawn scan failure (line 66) and the `go env` failure.
  - **Left unchecked, and unpinned by the fix:**
    - The `ScanTestFuncs` failure (line 53) has no row.
    - The `buildLym` failure (line 111) has no row.
    - Neither was named by the review, so this is not a defect of the fix.

Further calls made (3 of 6):
- `grep` for the `--max` counting statements across the repository's Go, markdown and YAML files, because my context held only the files the fixer touched.
- `grep` of `docs/overview.md` and `pattern/PATTERN-test-speed.md`, because they were not in my context.
- `sed` of `docs/overview.md:917-932`, to read the testmap paragraph in full.
