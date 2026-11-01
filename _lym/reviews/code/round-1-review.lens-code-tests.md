```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, from the `code-tests` lens.

Per behaviour, the test that pins it:

1. **The raw go steer offers each compile check only while no `refuseRaw` prefix refuses it** (`compileCheckSentence`, `refusedRaw`): pinned by `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` in `internal/shellguard/rows_test.go:95-124`.
   - The rows cover a nil list, `go test` refused (both checks offered), `go build` refused (only `go vet` offered), and `go` refused (no sentence, steer stays well formed).
   - Breaking the filter fails the NoRefuseList, GoBuildRefused and EveryGoCommandRefused rows.
   - Gap: no row has a prefix with more words than the form, for example `go build ./internal/x extra`. That is the only case that reaches the `len(prefixWords) <= len(words)` guard in `rows.go:229`. Removing the guard would panic and no test would catch it. See finding 1.
   - Gap: no row has a refused `go vet` with `go build` kept, so the second form's refusal is unpinned. It is symmetric with the `go build` row, so this is minor.
2. **The raw go notice is the steer plus a full stop, so it carries the compile sentence**: pinned by the `row.Notice != row.Steer+"."` assertion in the same test. The `Rows` call in `TestCheckTexts/ShippedRows` (`rows_test.go:34`) also keeps the gate rule in exactly one notice and runs `checkTexts` over the new text for forbidden characters.
3. **The offered `go build ./internal/x` and `go vet ./internal/x` are allowed by the raw-go deny pattern**: pinned by the two new rows `steered_package_scoped_build` and `steered_package_scoped_vet` in `TestRows_PatternsDenyAndAllow` (`rows_integration_test.go:98-99`). The module-wide counterparts `build_module_wide` and `vet_module_wide` at lines 78-79 are still denied.
4. **The python steer and notice carry the new wording**: the init-time forbidden-character check and the steer carrying the edit rule are pinned by `TestCheckTexts/ShippedRows` (`rows_test.go:41-46`).
   - The new text itself (the version-check clause and the jq, grep and awk parse routes) has no assertion. That is steer and notice prose, so it is exempt.
   - The wording only exists in the golden file (`settings-golden.json`), so any wording change is pinned only by that snapshot and the golden test fails if the render drifts.
5. **The claim that a version check is refused (`python3 --version` in command position)**: the python row's pattern is tested by existing rows, which I did not re-read. I found no row added in the diff for `python3 --version`. See finding 2.
6. **The regenerated golden**: pinned by the claudeengine golden test over `testdata/settings-golden.json` (the file is regenerated in the diff, and the render is compared to it).
7. **Doc and spec wording** (`doc.go`, `docs/overview.md`, `loom-plan-spec.md`): prose with no test, which is expected.

Findings:

1. **LOW, design**: `internal/shellguard/rows_test.go:95-124`. Add a row to `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` where a refuse prefix has more words than the form, for example `[]string{"go build ./internal/x extra"}`. The want string is the unchanged both-forms sentence. This pins the `len(prefixWords) <= len(words)` bound in `refusedRaw` (`rows.go:229`).
2. **LOW, scope**: the diff pins the claim "a version check is refused" only by prose, with no `python3 --version` row visible in `rows_integration_test.go`. The notice and steer now state that claim. Add a `python` row `python3 --version` with `true` to the pattern table in `TestRows_PatternsDenyAndAllow`, unless a row for it already exists there.

Tool calls: none. Everything came from the inherited context.
