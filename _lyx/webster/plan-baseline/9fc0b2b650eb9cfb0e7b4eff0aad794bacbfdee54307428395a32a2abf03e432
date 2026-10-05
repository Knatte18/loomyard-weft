# Card 13 — Retire the `smoke` tag: retag its files to `tmux` or `llm`

**Edit:**
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tmuxisolation_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/fabricengine/commit_lock_integration_test.go`
- `internal/gitkit/reexecguard.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/loomcli/smoke_gate_test.go`
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/orchcli/smoke_adopt_large_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/orchengine/doc.go`
- `internal/reedcli/cli_test.go`
- `internal/reedcli/smoke_attach_test.go`
- `internal/reedcli/smoke_coldstart_test.go`
- `internal/reedcli/smoke_debuglog_test.go`
- `internal/reedcli/smoke_dotfill_measure_test.go`
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_ifabsent_test.go`
- `internal/reedcli/smoke_lifecycle_test.go`
- `internal/reedcli/smoke_panecwd_test.go`
- `internal/reedcli/smoke_procalive_linux_test.go`
- `internal/reedcli/smoke_procalive_windows_test.go`
- `internal/reedcli/smoke_proctree_test.go`
- `internal/reedcli/smoke_removebyname_test.go`
- `internal/reedcli/smoke_resume_test.go`
- `internal/reedcli/smoke_selvage_keepalive_test.go`
- `internal/reedcli/smoke_staterecovery_test.go`
- `internal/reedcli/smoke_statuslineseed_test.go`
- `internal/reedcli/smoke_teardown_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/reedcli/smoke_warmpath_test.go`
- `internal/shedcli/parity_test.go`
- `internal/shuttlecli/modelpin_test.go`
- `internal/shuttlecli/smoke_bashstdin_test.go`
- `internal/shuttlecli/smoke_guardrail_test.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/shuttlecli/smoke_run_test.go`
- `internal/shuttleengine/claudeengine/startup_test.go`
- `internal/testkit/lyxbin/lyxbin.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/webstercli/smoke_test.go`
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-test-isolation.md`

**Uses:**
- `internal/testkit/llmkit#Claude`

**Intent:**
Every `smoke` file moves to `tmux` or `llm` (D6), and no `smoke` build tag remains.
File names keep their `smoke_` prefix: `internal/shuttlecli/modelpin_test.go` and the crucible docs find the files by it, and a rename would churn history for no behavior.

A file that spawns, or drives `lyx` into spawning, a real LLM session takes `//go:build llm`:
`internal/burlerengine/smoke_round_test.go`, `internal/burlerengine/smoke_cluster_test.go`, `internal/orchcli/smoke_cycle_test.go`, `internal/orchcli/smoke_live_test.go`, `internal/orchcli/smoke_adopt_large_test.go` (it resumes a real Claude Code session), `internal/reedcli/smoke_resume_test.go`, the four `internal/shuttlecli/smoke_*_test.go` files, `internal/treadleengine/smoke_judge_test.go` and `internal/webstercli/smoke_test.go`.
Every other `smoke` file takes `//go:build tmux`, platform terms kept (`smoke_procalive_linux_test.go`, `smoke_procalive_windows_test.go`): every `internal/loomcli/smoke_*` file drives a stub provider, and the remaining `internal/reedcli/smoke_*` files drive tmux alone.
Before committing, confirm each `tmux` file's tests reach no LLM: grep it for `claude` and for a `LYX_*_CLAUDE` override, and read any hit.

Each per-package LLM-binary lookup (`claudeBinaryPath` in reedcli, burlerengine and shuttlecli, `smokeClaudeBin` in webstercli, the `exec.LookPath("claude")` checks in orchcli and treadleengine) is replaced by `llmkit.Claude`, passing the override variable that copy honoured (`LYX_REED_CLAUDE`, `LYX_SHUTTLE_CLAUDE`, or `""` where it honoured none), and the local copy is deleted.
`internal/reedcli/smoke_test.go` is the shared harness of both tiers: its Claude-only helpers (`claudeProjectDir`, `claudeTranscriptFiles`, the transcript wait, `smokeClaudeModel`) move into `smoke_resume_test.go`, their one consumer, and the file itself takes `//go:build tmux || llm`, the shared-helper constraint recorded after Discussion.
Apply the same rule to any other helper file whose helpers both a `tmux` and an `llm` file use: it carries the disjunction of their tags, and a helper only one tier uses moves into that tier's file.
Each tag must compile on its own: `go vet -tags tmux ./...` and `go vet -tags llm ./...` each pass.

`cmd/lyx/tierpurity_test.go` and `cmd/lyx/tmuxisolation_test.go` drop `smoke` from their tag lists and tag sets, and their self-test rows for `smoke` become rows for `tmux` or `llm`.
Every comment, message or doc string in the listed Go files that names the `smoke` tag or `-tags smoke` names the new tier instead: `internal/orchengine/doc.go`'s recorded command becomes `-tags llm`, `internal/gitkit/reexecguard.go` and `internal/testkit/lyxbin/lyxbin.go` name `integration`, `tmux` and `llm`, and `internal/shuttlecli/modelpin_test.go` says its scanned files are `llm`-tagged.
The target list is every Go file outside `_lyx/` that names the tag: a `go:build smoke` line, `-tags smoke` or `-tags=smoke`, "smoke-tagged", "smoke tag", or `smoke` quoted or in backticks;
a hit that uses "smoke" as a plain word for a live test stays.
Card 16 updates the stencils, specs and docs that name the tag.

`PATTERN.md`'s `PATTERN-test-speed` line names the four tiers instead of "`integration` or `smoke`", and `pattern/PATTERN-test-speed.md` carries D6's tier table (tier, tag, needs, costs), the rules that tags do not nest, a test needing two substrates takes the higher tier, and each tag compiles on its own (with the disjunction rule for a shared helper), and that `llm` files are compiled by `go vet -tags llm ./...` and never run by a gate.
`PATTERN-test-isolation`'s line and background key "has a tagged test file" on `integration`, `tmux` or `llm`, and name the tag sets the check covers.

Bound: files moved to `llm` leave every automatic run, including `cardGateCommand`'s `-tags integration` run, and are run by hand only (D6);
files moved to `tmux` run under this plan's verify and by hand until note `test-tiers` decides gate selection.
No test body changes beyond the helper swap and the helper moves.

**ImpactSummary:** test files of nine packages plus three production doc comments, the two `cmd/lyx` guards and the Testing entries; no production behavior changes.
