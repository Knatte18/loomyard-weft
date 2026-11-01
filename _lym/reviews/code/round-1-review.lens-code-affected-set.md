```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change I looked for what reads or asserts the changed text.

**Per change, the set found**

1. **Python steer and notice wording** (`internal/shellguard/rows.go:45-52`).
   - Anything still asserting the old "lym agents never run Python" text? No. A repo-wide grep finds the old string nowhere outside the regenerated golden.
   - Consumers: `settings-golden.json` (regenerated), `TestCheckTexts/ShippedRows` (`rows_test.go:44`, still sees the edit rule in the steer), and `docs/overview.md`, `claudeengine/doc.go` and `shellguard/doc.go` (all updated). Covered.
   - `pattern/PATTERN-refusal-way-forward.md:18` lists `python` and `python3` as denied forms but quotes no wording. Unaffected.
   - The steer says "python and python3", while the row's pattern also refuses `python3.<minor>`. The overview and `claudeengine/doc.go` still state the full form, so nothing disagrees. Not a finding.
2. **Raw-go steer gains `compileCheckSentence`** (`rows.go:198-236`).
   - Consumers: `rows_integration_test.go` (new allow rows for `go build ./internal/x` and `go vet ./internal/x`), `rows_test.go` (new test), the golden, `claudeengine/doc.go:248`, `shellguard/doc.go`, and the overview paragraph. All updated. Covered.
   - `claudeengine/settings.go:466-472` renders the rows over the sample list `{"go test"}` for its init check. The steer contains no forbidden characters, so that check is unaffected.
   - `PATTERN-refusal-way-forward.md:20-22` says no steer names a raw module-wide `go` run as a way forward. The new sentence offers only package-scoped checks and says "never ./... or all". Compliant.
3. **`loom-plan-spec.md` compile criterion**: the whole-module `go build ./...` text is gone from criterion 1.
   - Other readers of the old wording: none in code, stencils or specs. The remaining `go build ./...` hits are in `planparser` test fixtures, `burlerengine/doc.go`, `gatecli/doc.go`, `crucible/*` and `cmd/lym/crosscompile_test.go`. None are steers or refusals. Covered.
4. **Declared PATTERN changes and commit trailers.** The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file. The single commit carries no `Pattern-Departure:` trailer and needs none.
5. **Stencil wording.** The diff changes no stencil.

**Findings:** none.

**Tool calls**
1. `grep -rln` over the shell-guard and python-deny symbols and the steer phrases, to find every reader of the changed text. My context held only the diff.
2. `grep -n` in `pattern/PATTERN-refusal-way-forward.md`, `internal/shuttleengine/doc.go`, `template.yaml` and `languageconfig/doc.go`, to check whether any pattern or doc quotes or constrains the changed wording. My context did not hold the pattern file.
