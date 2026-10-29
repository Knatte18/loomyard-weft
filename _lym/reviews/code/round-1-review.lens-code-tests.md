```yaml
lens: code-tests
outcome: reported
```

Lens list (code-tests), one line per behaviour the diff changes:

- **Darn step 4.3 and step 1 text (`darn-template-write.md`):** none, and none needed. This is static stencil prose, which the lens exempts.
- **Drift baseline step 1 pointer (`drift-template-baseline.md`):** none, and none needed. Static stencil prose.
- **Orch role placement sentence (`orch-template-role.md`):** none, and none needed. Static stencil prose.
- **Edit directive own-worktree rule (`edit-directive.md`, `internal/editdirective/doc.go`):** none, and none needed. The stencil text is static prose.
  - Unchanged behaviour: the directive is still rendered marker-free into every spawned role. `contracts/stencils/markers_test.go` and `contracts/stencils/parentdirective_test.go` cover that, and I did not read them.
- **Vocabulary scan root-file set (`enforcement_test.go:559`):** pinned by `fabricVocabularyFailures`, the scan that uses `vocabScanRootFiles`. With `CLAUDE.md` gone and dropped from the set, the scan covers `README.md` alone, and a stray `weft` or `warp` token there would fail it.
  - The rest of this file's diff is comment-only.
- **`merriamFixedContext` (`merriambase.go`):** the value is unchanged and only the comment changed, so there is no new behaviour. `merriambase_test.go` pins the line sum and fixed context from files it plants in a temp worktree, so it does not depend on the repo-root `CLAUDE.md`.
- **Comment-only edits (`crosscompile_test.go`, `reflow.go`, `fabricengine/doc.go`, `PATTERN-fabric-vocabulary.md`):** no behaviour changes, so no test is needed.
- **Deletion of repo-root `CLAUDE.md`:** my grep of `tools/` and `cmd/` Go files found no code reading it. The `merriambase_test.go`, `webstercli/cli_test.go` and `runlevel_test.go` tests that name `CLAUDE.md` plant their own copy in a temp worktree, going by the earlier grep output in context. None of them reads the repo's file.

Findings: none.

Tool calls:
1. `grep -rn "CLAUDE\.md" tools cmd --include='*.go'` (excluding `tools/mdreflow`): to check that no tool or cmd code reads the deleted file. The earlier grep output was truncated by `head -80` before reaching `tools/`.
