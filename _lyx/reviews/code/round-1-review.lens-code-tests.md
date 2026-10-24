```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: list of behaviours the diff adds or changes, and the test that pins each.

- Test-failure messages and comments re-pointed to survivor IDs or package docs, in the 27 `_test.go` files that changed. No behaviour changed. The tests' own assertions are unchanged, so each still fails when its rule breaks, and the new message text is not itself pinned. Pinned by the same enforcing test as before; no new test is needed.
- `tools/sandbox/pathresolve_guard_test.go`: only the header comment and the error message changed. The rule stays pinned by `TestPathResolveGuard_NoBarePathLyxOutsideResolve`.
- `tools/sandbox/doc.go` and the other `doc.go` additions: documentation only, no behaviour added. The rules they state keep their existing enforcing tests (`cmd/lyx/sandbox_coverage_test.go` for suite coverage, the path-resolve guard for binary separation), and none was removed.
- `PATTERN.md` and `pattern/` merges and deletions: the checker `internal/pattern.Check` pins one background file per linked entry. I did not read its test. The handler or the affected-set lens should confirm it passes.
- Test economy: the diff adds no `func Test` and no table row, and removes no test file (`--diff-filter=AD` over `*_test.go` is empty). No duplicate test, per-helper test, review-fix test, stencil-prose assertion or uncovered added behaviour.

Findings: none for this lens.

Tool calls (1 of 6):
1. `git diff ed98ccf71..HEAD -U0 -- '*_test.go'` filtered for added or removed `func Test` and table rows, plus the list of added or deleted test files. My context showed only the message changes in the filtered diff, so I confirmed that no test was added or removed.
