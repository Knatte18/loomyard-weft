---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: tools/godocreflow/reflow_test.go:35
    summary: two new test doc comments break mid-sentence, against PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/selfreportengine/selfreport_test.go:117
    summary: TestCreateIssue_Success and its sibling tests swap the package-level NewGitHubClient seam without t.Parallel or a comment naming that state
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/loomshed/seed_test.go:15
    summary: TestSeed_WritesExpectedStatus calls t.Parallel per subtest but not at the top level, and touches no global state
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/test-suite-prune-rest/status.json` names `main` as `product.parent`, and `_lyx/loom/rework/` does not exist.
The three focus sweeps were run mechanically over the whole range.

- Sweep 1 used an AST scan of every added or modified `_test.go` file for top-level tests whose range or for loop calls `t.Run`, flagging a missing top-level or per-subtest `t.Parallel`.
  The scan's output was compared with base and then read by hand against each file's header and test-level comments.
- Sweep 2 extracted every comment line the diff adds and flagged consecutive added lines where the first does not end a sentence and the next begins lowercase, dropping lines byte-identical to a base comment line.
- Sweep 3 ran `golangci-lint` with the `unused` linter over `integration,tmux,llm`, and compared the top-level test names of base and HEAD against every `Test…` name mentioned in any `.go`, `.md` or other tracked file.

The scan's comment check matched only keywords such as `parallel` and `global`, so it first reported two more tests (`TestLoadConfig_ScalarKeysAndTemplateDefaults` and the two `landingshed` commit-status tables); a read of their doc comments showed each already says why it runs serially, so those candidates are withdrawn and never became findings.

Results of the sweeps and checks that are not findings:

- Every other serial table in the focus packages (`landingshed` `finalize`, `finalize_verify`, `prgate`, `publish`, `publish_verify`, `internal/logger`, `cmd/lyx/exitsweep_test.go`, `tools/sandbox`, `reedcli`, `cliwire`, `githubclient`, `selfreportcli`, `envsource`) names its global state in a test or file-header comment, and the scenarios (`TestSync`, the `gitrepo` parity and push scenarios, `TestDirective_PerRole`, `TestParseOwnerRepo`) carry the reason their steps run serially.
- No top-level test deleted or turned into a subtest by the prune is named in any file at HEAD, so there is no dangling reference; the `docs/benchmarks/test-suite-timing.md` rows that name base-only tests are a measurement report and stay as recorded.
- `unused` reports no test helper without a caller in `landingshed/prgate_test.go` or `reedcli/smoke_test.go`; the unused `PRGate` field `gate` and the `beforeCreate` field exist unchanged on base, so they are not this diff's leftovers.
- `go build ./...`, the four `go vet` tag variants and `go test -count=1 ./...` pass.

### [BLOCKING] Mid-sentence line breaks in new godocreflow test comments

**Location:** `tools/godocreflow/reflow_test.go:35`, `tools/godocreflow/reflow_test.go:135`
**Issue:** The doc comments on `TestReflowSource_ReflowsDocComments` and `TestReflowSource_LeavesUntouched` break inside their sentences ("an" / "exported func", and ", and the width fallback" on its own line), against `PATTERN-comment-line-breaks`.
**Fix:** Rewrite both comments with one sentence per line and no break inside a list.

### [BLOCKING] Serial selfreportengine tests name no reason

**Location:** `internal/selfreportengine/selfreport_test.go:117`
**Issue:** `installGitHubClient` replaces the package-level `NewGitHubClient`, so no test here can run in parallel, but neither the file header nor the table tests say so.
**Fix:** Add a sentence to the file header naming the seam swap as the reason no test calls `t.Parallel`.

### [BLOCKING] Missing top-level t.Parallel in the loomshed seed table

**Location:** `internal/loomshed/seed_test.go:15`
**Issue:** Each subtest calls `t.Parallel`, but the top-level test does not, and the rows touch nothing but `t.TempDir()`, so no global state forbids it, against `PATTERN-test-speed`.
**Fix:** Call `t.Parallel()` at the top of `TestSeed_WritesExpectedStatus`.
