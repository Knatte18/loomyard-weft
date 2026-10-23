# Card 3 — test-file-patterns

**Create:**
- `plan:internal/planparser#testFileMatchers` -> `var testFileMatchers map[string]func(path string) bool`
- `plan:internal/planparser#IsTestFile` -> `func IsTestFile(language, path string) bool`
- `plan:internal/planparser#IsAnyLanguageTestFile` -> `func IsAnyLanguageTestFile(path string) bool`
- `plan:internal/planparser#PlanLanguage` -> `func PlanLanguage(plan *Plan) string`
- `plan:internal/planparser#TestIsTestFile` -> `func TestIsTestFile(t *testing.T)`

**Extends:**
- `internal/planparser/doc.go`

**Uses:**
- `internal/planparser#sourceExtensions`
- `internal/planparser#effectiveLanguage`

**Intent:** Declare the per-language test-file patterns the fixer caps read (D4, D10), beside `sourceExtensions` in `reviewclass.go`, without sharing its default.
`testFileMatchers` maps `go` to a file whose name ends in `_test.go`, and `csharp` to a `.cs` file whose name ends in `Tests.cs` or `Test.cs` or that lies under a directory whose name ends in `Tests`; paths are slash-separated and repository-relative.
`IsTestFile(language, path)` answers the language's matcher, and a language the table lacks, `none` included, answers true for every path, so a cap over-covers rather than lets a test edit through.
`IsAnyLanguageTestFile(path)` answers true when any declared language's matcher matches, the union a darn run takes since it has no plan.
`PlanLanguage(plan)` exports `effectiveLanguage`'s answer, the plan's `language:` with an absent key read as `go`, for the loom wiring that tells the caps their matcher.

The table-driven test pins each row: `foo_test.go` and `foo.go` under `go`, `FooTests.cs`, `FooTest.cs`, `src/BarTests/Baz.cs` and `src/Bar/Baz.cs` under `csharp`, any path under `none` and under an unknown language, and the union for a `_test.go` file, a `FooTests.cs` file and a plain `.go` file; it also pins `PlanLanguage` for an absent, a `go` and a `csharp` key.

`doc.go` names the table and its over-covering default.

**ImpactSummary:** Additive exported functions and one doc paragraph; `sourceExtensions` and every existing reader are untouched.
