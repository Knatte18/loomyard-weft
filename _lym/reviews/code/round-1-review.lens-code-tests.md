```yaml
lens: code-tests
outcome: reported
```

Task: as the code-tests fork, for each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **The changed lines of all 130 `doc.go` files (package doc prose only):** none. They have no runtime behaviour, so no test row can pin them. This is exempt: the rubric exempts static prose, and the focus file confirms the diff changes no code or tests.
- **The `// Package <name>` doc comment staying attached to each package clause:** none pins it. That is the comment lint's subject (`lym loom lint-comments`), not a test's, and the handler's manifest check found no detached clause.
- **fabricengine's `doc.go` naming its Mutation Record Invariant guard test:** `TestMutationRecord_FabricengineProductionSource`, marked `//testtiming:keep` at `cmd/lym/destructiveguard_test.go:248`. The doc still names it at `internal/fabricengine/doc.go:314`. The test checks the source of the executors and result types, not the doc text.
- **fabricengine's `doc.go` as an allowlist entry in the guard scans** (`cmd/lym/destructiveguard_test.go:115`, `cmd/lym/uncontainedwrite_test.go:80`): these are scanner exemptions, and no test asserts doc prose. Whether the exemption reasons are now stale is code-affected-set's subject, not this lens's.
- **No other test reads a changed `doc.go` file's text.** The other `_test.go` hits on `doc.go` paths use synthetic fixture paths (`internal/loomshed/gates_test.go:1324-1364`, `internal/planparser/*_test.go`, `tools/plandag/main_test.go`).

**Findings:** none. The diff adds or changes no behaviour, and it adds, removes or loosens no test. The redundancy, reach and mutation reports are empty, which matches a comment-only diff.

**Tool calls**

1. `git grep` for `doc.go` references in `*_test.go`: to find any test that reads `doc.go` text. My context only had the fabricengine guard lines, not a full sweep.
