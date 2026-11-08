```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each as-proposed fix through the fix-proposal lens. I made no tool calls and relied on the context I already held.

**Findings**

1. MEDIUM — `internal/impactset/mains_integration_test.go`, subtest "a main package that fails to build stays listed" (F5).
   - **Summary:** the proposed "type error" fixture does not exercise `-e`, so the test passes with `-e` removed.
   - **Evidence:**
     - The fixture is `func main() { var count int = "text" }`. `go list` only loads packages: it reads file headers and imports, and never type-checks.
     - The type error never reaches `go list`, which exits 0 and prints the package with or without `-e`. The subtest cannot fail if `-e` is dropped from `mains.go:32`.
     - The tier-1 pass in the fixer's own run is consistent with that. It shows the subtest passes with `-e` in place, not that it pins `-e`.
     - `-e` only matters for load errors. Examples are an import of a package that does not exist, such as `import _ "example.com/m/missing"`, or a file that fails to parse. Without `-e`, `go list` exits non-zero on these, and `ModuleMainPackages` returns an error.
   - **Better change:** use a missing-import (load-error) fixture, or a syntax error that `go list` reports, such as `package main\n\nimport _ "example.com/m/nosuch"\n\nfunc main() {}\n`. Then confirm the subtest fails when `-e` is removed.

2. LOW — `internal/gatecli/scopes_integration_test.go`, rows "adds nothing under the none scoper" and "adds nothing under dotnet" (F4).
   - **Summary:** the two rows are weak regression pins.
   - **Evidence:**
     - Under `none`, `computeScope` returns `whole` at the `cfg.Scoper != "go"` branch before any main-packages code runs. The row asserts `./...`, which is the same result with or without `--main-packages`.
     - Under dotnet, `dotnetScope` never reads `request.mainPackages`. The row asserts the same `circleTestsArgs` as the existing dotnet row.
     - The rows only fail if someone moves the main-packages logic ahead of the scoper dispatch. They do pin the "adds nothing" contract the finding named, so this is acceptable but weak.

3. LOW — `contracts/stencils/webster/webster-body-implementer.md:131` (F3).
   - **Summary:** line 131 still reads unqualified: "A failure the card did not cause is never yours to fix".
   - **Evidence:**
     - Line 107 now names this self-fix-bound text as an exception. Line 132 and line 175 carry the explicit "except" clause, but line 131 itself does not.
     - A reader who stops at line 131 gets the old rule. The reviewer's fix key only asked for the two "FAILED" sentences, so this is a residual sibling rather than a defect in the fix.

**Not findings**
- F1: batch 2 is the last batch of the fixture's two. The slug `2-list-tests` matches the recover test's pattern. The `wantNot` string `--card 1-json-flag --main-packages` is not matched by the stencil prose, which only has `--main-packages` after a backtick.
- F2: the exact-equality rows (`slices.Equal`) catch both dropping and duplicating the word.

Tool calls: none.
