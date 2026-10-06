---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/finalize_test.go:313
    summary: Table-driven tests written by the fold step carry no t.Parallel at top level or per subtest and no comment naming the global state that forbids it, against the plan's prune-procedure step 5 and PATTERN-test-speed
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/battencli/lifecycle_integration_test.go:161
    summary: New scenario, step and fold comments are fixed-column hard-wrapped mid-sentence, against PATTERN-comment-line-breaks
  - id: F3
    severity: MEDIUM
    class: scope
    location: pattern/PATTERN-batten-bookend.md:11
    summary: The background file names TestBattenIntegration_RecordsPrimeRefusal, which the batten scenario merge turned into a step of TestBattenIntegration_Rows, so the named top-level test no longer exists
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/landingshed/prgate_test.go:76
    summary: Pruning left test helpers with no caller under any tag, against prune-procedure step 7
---

### [BLOCKING] Fold tables lack t.Parallel and a global-state comment

**Location:** `internal/landingshed/finalize_test.go:313` (`TestFinalize_StuckReasons`) and the other new-named table or fold tests listed below.
**Issue:**
Decision `prune-procedure` step 5 requires a folded table to use `t.Parallel` "at the top level and per subtest unless a comment names the global state", and `PATTERN-test-speed` says `t.Parallel` runs unless a comment names global state.
An AST scan of every test function that runs `t.Run` inside a range loop, restricted to test names that did not exist on the base commit `9cd12fd19`, finds tests with no top-level `t.Parallel()` and no doc or inner comment that names a reason.
Some of them also lack the per-subtest call (those with a subtest-level call but no top-level call are in the list too).
The package convention before the task was non-parallel in several packages (`landingshed` had no `t.Parallel` at the base), but the plan's decision binds the fold output, not the package's history.
Where a test truly cannot be parallel (`t.Setenv`, `t.Chdir`, a package-level stub such as `stubResolveLyxProd`, or the process-global logger), the comment naming that state is the missing piece; elsewhere the call is missing.
The list, as `file (count)`:
`cmd/lyx/exitsweep_test.go` (2), `internal/battencli/commitstatus_test.go` (1), `internal/githubclient/githubclient_test.go` (1), `internal/landingshed/finalize_test.go` (6), `internal/landingshed/finalize_verify_test.go` (2), `internal/landingshed/publish_test.go` (7), `internal/landingshed/publish_verify_test.go` (3), `internal/logger/logger_test.go` (2), `internal/logger/sink_test.go` (2), `internal/logger/span_test.go` (1), `internal/logger/trace_test.go` (1), `internal/loomshed/gatefindings_test.go` (1), `internal/orchengine/launch_test.go` (1), `internal/orchengine/watcher_compact_test.go` (1), `internal/orchengine/watcher_test.go` (1), `internal/orchengine/watchloop_test.go` (1), `internal/reedcli/cli_test.go` (1), `internal/reedcli/removename_test.go` (1), `internal/reedengine/render/height_test.go` (1), `internal/selfreportcli/cli_test.go` (3), `internal/summaryparser/summary_test.go` (1), `internal/webstercli/cli_test.go` (2), `tools/sandbox/main_test.go` (4), `tools/sandbox/report_test.go` (1), `tools/sandbox/suite_test.go` (2).
**Fix:**
For each listed test, add `t.Parallel()` at the top level and in each subtest when nothing global is touched, and otherwise add a comment naming the global state that forbids it.
Prove each added call with `go test -race -count=1` on the package.

### [BLOCKING] Hard-wrapped new comments

**Location:** `internal/battencli/lifecycle_integration_test.go:161`, `internal/logger/sink_callsite_integration_test.go:67`, `internal/configcli/configcli_integration_test.go:29`, `internal/configcli/reconcile_integration_test.go:56`, `internal/fabriccli/cleanhub_integration_test.go:96`, `internal/fabriccli/cleanupremove_integration_test.go:90`, `internal/boardengine/sync_integration_test.go:85`, `cmd/lyx/stencilseed_integration_test.go:59`, among many.
**Issue:**
`PATTERN-comment-line-breaks` requires one sentence per line with no column limit, for every new or changed comment.
The diff adds comment lines that break mid-sentence at roughly 100 columns, for example the scenario headers that explain why a scenario is serial and which step relies on which.
A comparison of the added comment lines against every comment line of the base tree finds many such lines that did not exist before, so they are new text rather than moved legacy text.
**Fix:**
Rewrite each newly added comment block so each sentence sits on its own line, with a further break only at an independent-clause boundary.
Leave comment lines that are byte-identical to base text untouched.

### [MEDIUM] Stale test name in a pattern background file

**Location:** `pattern/PATTERN-batten-bookend.md:11`
**Issue:**
Decision `prune-procedure` step 7 requires a doc or `pattern/` background that names a deleted or renamed test to be updated in the same commit.
`TestBattenIntegration_RecordsPrimeRefusal` is now the step `RecordsPrimeRefusal` of `TestBattenIntegration_Rows` in `internal/battencli/lifecycle_integration_test.go`, so no top-level test of that name exists.
A repository-wide search of every non-benchmark file for each removed top-level test name finds only this reference.
**Fix:**
Name `TestBattenIntegration_Rows/RecordsPrimeRefusal` in the sentence.

### [MEDIUM] Helpers left with no caller

**Location:** `internal/landingshed/prgate_test.go:76` (`wantVerdict`), `internal/reedcli/smoke_test.go:720` (`mustChdir`) and `internal/reedcli/smoke_test.go:782` (`paneIDForStrand`)
**Issue:**
Decision `prune-procedure` step 7 deletes a test helper no file calls under any tag.
Under `-tags integration,tmux,llm` the `unused` analyzer reports all three, and each had callers on the base commit that the prune removed.
Pre-existing unused symbols (`prGateFixture.gate`, `publishFixture.beforeCreate`) and unused symbols in core packages are not this task's.
**Fix:**
Delete the three helpers.

## Checks that found nothing

- No edit under `internal/fabricengine`, `internal/websterengine`, `internal/reedengine` outside `render`, `internal/shuttleengine` outside `claudeengine`, `internal/loomcli`, `internal/shedadapters`, `internal/testkit/` or `cmd/testtiming`; the only production edit is a comment in `internal/stencil/stencil.go` that named two folded tests.
- Skill-loading tests (`TestSkillLoadSequence`, `TestSkillUnknown`, `TestPrepare_SkillsDeferPromptPointer`) keep their names and none is a redundancy candidate, so no `skills-in-one-turn` keep is owed.
- Every one of the 269 `//testtiming:keep` directives sits directly above a `func Test` line with a reason of one of the plan's classes, and every in-scope candidate of the committed report carries one.
- No test named in a core or testkit file was removed or renamed; the sections of `docs/benchmarks/test-redundancy.md` for the core packages are unchanged.
- `go build`, `go vet` under the default, `integration`, `tmux` and `llm` tags, and `go test -count=1` under the default, `integration` and `tmux` tags all pass on the final tree.
- Card 1's commit touches only `docs/benchmarks/`, so the before measurement was taken on the untouched tree; the after section and the rerun splice are consistent with it.
- The reedcli smoke files carry `tmux || llm` and import no `llmkit`, so they are tmux-tier files, not `llm` files.
