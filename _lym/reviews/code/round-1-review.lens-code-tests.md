```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **Finding text names `lym loom fmt <file>` as the fix** (`internal/testkit/gofmt_test.go:58`). The row "unformatted source names file, line, current and formatted text" now expects `"lym loom fmt a.go"` (`internal/testkit/gofmt_test.go:76`). Drop the hint, or put the wrong path in it, and that row fails. Pinned.
2. **Finding text still names the file, line, current text and formatted text.** The rows "unformatted source…" and "import order alone still fails with that line" cover this (`internal/testkit/gofmt_test.go:76,79`). They carry over unchanged from `cmd/lym/gofmt_test.go:66,69`. Pinned.
3. **An unparseable source is a finding and a formatted source passes.** The rows "unparseable source names the file" and "formatted source passes" cover this (`internal/testkit/gofmt_test.go:77-78`). They carry over unchanged. Pinned.
4. **The walk started from `internal/testkit` still reaches the whole module.** The walk resolves its root through `scankit.ModuleRoot` from the test binary's starting directory (`internal/testkit/scankit/scankit.go`, `startDir`/`findModuleRoot`). If the root resolved wrong, too few files would be scanned and `scankit.RequireFloor(t, scanned, gofmtScanFloor, "gofmt scan")` would fail (`internal/testkit/gofmt_test.go:101`, floor 500 at `:35`). Pinned.
5. **The guard runs at every task's tier0 gate now that `//lym:only parent-align finalize` is gone.** This comes from the markers, not from code in this diff. The existing gate tests cover how `//lym:guard` and `//lym:only` are handled (`internal/impactset/guards_test.go`, `internal/impactset/only_test.go`, `internal/gatecli/scopes_integration_test.go`). A test asserting this one test's marker set would only pin static text, so no new test is owed. Covered by the existing gate tests.
6. **`lym loom fmt` help text and the `internal/testkit` package doc name the new location** (`internal/loomcli/fmtverb.go:105`, `internal/testkit/doc.go:14`). This is static prose, so it needs no test.

**Findings:** none. No assertion was loosened: all four rows and the floor carry over at the same strength, and the new row assertion only adds to them. No new test duplicates one that already exists.

Tool calls: none. My inherited context already held the diff pieces, `scankit.go` and the gate and impactset code.
