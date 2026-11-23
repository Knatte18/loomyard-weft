```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped the diff's changes to the board entry's goal points.

**Decisions (the decision record alone is the answer key)**
- G1, from `## Goal`: "lint the working tree, or add a flag that does, or document the HEAD-only behaviour in the verb's help." It offers three alternatives, and building any one of them satisfies it.
- The record has no `Added after Discussion` entries.

**Per change**
1. `commentlint.Lint` now always diffs and reads the working tree against the base, and an empty base becomes HEAD (`internal/commentlint/lint.go` Lint/lintChanges; `internal/commentlint/diff.go` changedGoFiles/readChange take `workingTree`). This traces to G1, first alternative.
2. The new `commentlint.LintCommitRange` and the verify gate's switch to it (`internal/commentlint/lint.go` LintCommitRange, `internal/loomshed/verifygate.go:146`). These are forced by change 1. Without them the Webster-Burler-Fix verify gate, which passes a non-empty `lintBase`, would start reading the working tree, so the split keeps that caller's base-to-HEAD behaviour.
3. Doc and help text: `internal/commentlint/doc.go`, the diff.go file header and godocs, the `internal/loomcli/lintcomments.go` Long help and `--base` flag text, `docs/overview.md:1303-1308` and `pattern/PATTERN-gate-slot.md:64-65`. These are forced by changes 1 and 2, because each described the old HEAD-only reading of `--base` or listed the verify gate as a `commentlint.Lint` caller.
4. Tests: `internal/commentlint/lint_integration_test.go` (new subtests for both functions, reordered setup) and `internal/loomcli/lintcomments_integration_test.go` (the new `--base` uncommitted-edit subtest). These are forced by changes 1 and 2.

**Per decision**
- G1 is built by change 1, with changes 2, 3 and 4 forced by it.

**Findings:** none. Every change traces to G1 or is forced by a traced change, and G1 is built. `lint-pattern --base` is left unchanged (summary.md "Design decisions"). That is in scope, since the goal names only `lint-comments`.

Tool calls: none.
