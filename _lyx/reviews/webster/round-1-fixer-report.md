# Round 1 fixer report

All four findings are fixed, one commit each.
`go build`, `go vet` under the default, `integration`, `tmux` and `llm` tags, and `go test -count=1 ./...` pass after the last commit.
After the last commit `go test -count=1 -tags integration` was rerun for `cmd/lyx`, `internal/landingshed` and `internal/battencli` and passes; the `tmux` tier was not rerun, because the fixes after the full-tree run touch only comments, unit-tier tests and unit-tier helpers.

## Fixed

### F3 — stale test name in `pattern/PATTERN-batten-bookend.md` (`ff17300e2`)

The sentence now names `TestBattenIntegration_Rows/RecordsPrimeRefusal`.

### F4 — helpers with no caller (`61cc4c021`)

Deleted `wantVerdict` in `internal/landingshed/prgate_test.go`, and `mustChdir` and `paneIDForStrand` in `internal/reedcli/smoke_test.go`.
The doc comment of `paneIDForStrandIn` no longer refers to the deleted `paneIDForStrand`.
`golangci-lint`'s `unused` under `-tags integration,tmux,llm` no longer reports them.

### F1 — fold tables without `t.Parallel` (`1725a743d`)

The review's list was too wide: its scan read only function-level comments, and most listed tests sit in a file whose header comment already names the global state that keeps every test of the file serial (`t.Setenv`, `t.Chdir`, the package-level `NewGitHubClient` seam, the process-global logger, the sandbox seams).
Those tests satisfy the plan's "unless a comment names the global state" and needed nothing.
The tests with no such comment were fixed:

- `t.Parallel` at the top level, and per subtest where missing: `TestNewCommitStatusSeam`, `TestChooseStartPrompt`, `TestCompact_RestartWithBoundaryReadTypesNothing`, `TestWatcher_FailingRenderAbortsWithoutClear`, `TestRun_NoticeQueueAtExit`, `TestStackHeights`, `TestParse` in `internal/summaryparser`, `TestValidateCmd_Envelopes` and `TestBringUpDisposition_RecoverAndRun`.
  `go test -race -count=3` passes on the five packages that hold them.
- `TestBatcherRows_StuckSurfacesTheBatcherError` stays serial and its doc comment now names the reason: `logcapture.Capture` redirects the process-global logger output.

### F2 — hard-wrapped new comments (`d9f483deb`)

A script rejoined and re-split, one sentence per line, every comment block that carries a line absent from the base tree's comments, in the files the task changed.
Only comment lines changed: the non-comment lines of each file are byte-identical before and after, `//go:`, `//testtiming:` and `//tiermove:` lines are unchanged and still sit directly above their `func`, and the words of each block are identical.
Comment lines untouched by the task keep their wrapping.
`tools/godocreflow/reflow_test.go` was left out: its `//` lines inside raw-string fixtures are test data, not comments.
One list item in `contracts/stencils/citation_enforcement_test.go` that the script did not rejoin was joined by hand.
A clause that follows a colon at a line end stays on its own line, as a clause-boundary break.

## Deferred

Nothing.
