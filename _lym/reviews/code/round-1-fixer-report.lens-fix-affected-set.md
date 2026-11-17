```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of the sets behind F1, F2 and F3.

F1 (the sites that describe the `code-scope` lens or how it reads a decision record):
- The only definitions are `internal/lenses/template.yaml` and `_lym/landing/summary.md`, and the fix changed both.
- Other mentions of `Added after Discussion` are stencils, specs and tests for the record format, not descriptions of this lens.

F2 (the sites that list the `review-code` fan):
- Every list is covered: `template.yaml:424`, `sets/csharp.yaml:49-54` (already holds `code-scope`), `docs/dotnet-hub.md` (now fixed) and the assertions in `config_test.go`.
- No other doc repeats the fan.

F3 (the tests that assert the lens prose):
- The only such site was the `code-scope` row in `config_test.go`, which was deleted.
- Nothing else asserts the lens prose.

Findings left unchecked: none.

no findings

Tool calls:
1. grep for `code-scope` and `Added after Discussion` across the repo, to find every description of the lens and of the record. My context had only the files the fixer read.
2. grep for `code-tests` across the repo, to find every fan listing and every test that names the lens. My context had only `docs/dotnet-hub.md` and `config_test.go`.
3. grep on `sets/csharp.yaml` with context, to confirm the csharp fan holds `code-scope`. My context lacked that file.
