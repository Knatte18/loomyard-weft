```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none.

**Per behaviour: test that pins it**
- `stencilstore.ReadSet` reading `engine/<engine>/<block>.md`, the plain-include passthrough, the missing-block error naming block, engine and file, the empty-engine refusal and the unreadable stencil: `TestReadSet` (`engineblock_test.go:610`). Pinned.
- `RelPath` for a name holding `/`: the new `TestRelPath` row (`stencilstore_test.go:202`). Pinned.
- The shipped registry accepting `engine/<engine>/<block>` names: the `registry_test.go` helper change in `TestRegistry_MatchesOnDiskTree`. Pinned. No engine blocks ship yet, so the new branch has no data.
- `seatengine` `Table.Validate` reading a seat's blocks for its own engine or the default engine, the missing-block refusal and the empty-engine refusal: five new `TestTable_Validate` rows (`table_test.go:520-528`). Pinned.
- `Table.CheckMarkers` reading engine blocks per seat engine: two new rows (`table_test.go:555-562`). Pinned.
- `composeSeatPrompt` and `Prompts` composing with each seat's engine, else `geom.DefaultEngine`: `TestPrompts_RenderEachSeatsEngineBlocksForItsEngine` (`prompt_test.go:444`). Pinned.
- `Engine.Run`, `Probe` and `Resume` passing `e.geom.DefaultEngine` to `Validate`, and `startSeat` passing `e.geom` to `composeSeatPrompt`: **none**. See F1.
- `burlerengine` `composeReviewPrompt` and `composeFixPrompt` rendering a stencil's engine block for the given engine: `TestComposePrompt_RendersEngineBlocksForTheHalfsEngine` (`template_test.go:98`). It covers the review-step and fix-step stencils only.
- `burlerengine` `Engine.seatEngine`, which picks `choice.Engine` else `geom.DefaultEngine`: **none**. See F2.
- `Review` and `Fix` passing `opts.Review.Engine` and `opts.Fix.Engine`, and `New` copying `DefaultEngine` into the seat geometry: **none**. See F2.
- An engine block in the explore stencil, the focus directive, the fork-reading and cluster parts, the self-check part and the review-format stencil: not tested through an engine block. Every path goes through `fillInstruction` or `ReadSet` with the same engine argument. The explore stencil's own `ReadSet` call is the one site that is not covered. See F3.
- `DefaultEngine <- shuttleCfg.Engine` in `burlercli` (hub and standalone) and `loomcli` (the burler geometry, the MultiLLM `Seats` geometry and `shedrecipe.Env`), `Env.DefaultEngine` into `entries_multillm.go`'s `Validate`, and `darnwiring.go`'s `CheckMarkers(…, c.shuttleCfg.Engine)`: **none**. See F4.
- `heddleengine` `codex-file` opening with `AGENTS.md`, else `CLAUDE.md`, else nothing, and `AGENTS.md` winning over `CLAUDE.md`: four `CodexFileOpensWithRootAgents` subtests (`render_test.go:216-246`). Pinned.
- Repository `claude-file` entries carried into `codex-file`: `render_test.go:234-241`, which asserts `AGENTS.override.md == Marker\n\none\n\ntwo\n`. Pinned.
- Shipped entries kept off `codex-file`: the same assertion. It holds no shipped text, and the real shipped layer targets `claude-file`, so a wrongly included shipped entry would show. Pinned.
- A render whose target list names `claude-file` also rendering `codex-file`: the `OnlyLimitsTheTargets` rows `CodexFileAlone` and `ClaudeFileRendersCodexFileToo`. Pinned. Both rows run on a fresh fixture, so "a worktree that already exists takes a changed repository entry" has no separate row; it is the same code path.
- A `CLAUDE.md` change re-rendering `codex-file`: the `seam_test.go` rows `ClaudeMd`, `AnchoredClaudeMd`, `ClaudeDirClaudeMd` and `ClaudeMdAboveTheAnchor`. Pinned.
- The `claudeFileTarget` and `codexFileTarget` constants and the `withCodexFile` helper: covered through `Render`. The helper needs no test of its own.
- The `table.Validate(…, "")` signature follow-through in the `darnengine`, `driftengine`, `landingshed` and `loomengine` tests: mechanical, no new behaviour.
- Test economy: I found no duplicate of an existing test and no new assertion on static stencil prose. The test-only stencils (`engine-test-words`, `engine-test-marked`) are temp-dir fixtures, not shipped prose. The `TestReadSet` row `EngineBlockWithNoEngineIsRefused` and the `TestTable_Validate` row for the same case test one behaviour at two layers (the resolver, then the table's wrapping).

**Findings**
- **F1** MEDIUM, design. Location: `internal/seatengine/engine.go:230`, `resume.go:30` and `resume.go:94`. `Run`, `Probe` and `Resume` pass `e.geom.DefaultEngine` to `Validate`, and `startSeat` passes `e.geom` to `composeSeatPrompt` (`engine.go:293`). No test constructs an `Engine` with `Geometry.DefaultEngine` set and drives these. A regression passing `""` would still pass every test and would refuse every engine-block table at run time. The focus asks for the engine to be passed at every read of a seat's stencil set. `Prompts` and `Validate` are pinned directly, but the engine entry points are not. Evidence: the test diff holds only direct `Validate`, `CheckMarkers` and `Prompts` calls.
- **F2** MEDIUM, design. Location: `internal/burlerengine/engine.go:56` and `engine.go:73-79`, `review.go:99`, `fix.go:157`. Nothing tests `seatEngine`'s fallback (`choice.Engine` else `Geometry.DefaultEngine`). Nothing tests that `Review` passes `opts.Review` and `Fix` passes `opts.Fix`. Nothing tests that `New` hands `DefaultEngine` to the seat geometry. The new test calls `composeReviewPrompt` and `composeFixPrompt` with a literal engine string. Swapped `opts.Review` and `opts.Fix`, or a dropped fallback, would go unseen. Evidence: `template_test.go:130-134`.
- **F3** LOW, scope. Location: `internal/burlerengine/prompt.go:116`. The engine-block test appends the block include to `burler-step-review` and `burler-step-fix` only. The explore stencil reads its blocks through a separate `ReadSet` call, and no row covers an engine block there. Fix: add `burler-step-1-explore` to the map in `TestComposePrompt_RendersEngineBlocksForTheHalfsEngine`, with `Files[exploreStepFile]` asserted.
- **F4** LOW, scope. Location: `internal/burlercli/wiring.go:162` and `wiring.go:234`, `internal/loomcli/wiring.go:565`, `741` and `910`, `internal/loomcli/darnwiring.go:272`, `internal/shedrecipe/entries_multillm.go:98`. These wirings that carry shuttle's default engine have no test. A dropped assignment compiles and leaves `""`, and `ReadSet` then refuses any engine-block stencil. No shipped stencil uses an engine block yet, so nothing fails today. Fix: a test per wiring seam where one is cheap, such as the `shedrecipe` MultiLLM entry validating an engine-block stencil under `Env.DefaultEngine`.

**Tool calls**
1. Read `/home/knatte/.claude/projects/…/tool-results/b5zi1ybsa.txt`, the persisted test diff, because my context did not hold it.
2. One Bash call running `grep` for shipped-layer handling in `render_test.go`, `ReRender` use in `heddleengine/*.go`, and `DefaultEngine` in `*_test.go`, to check shipped-entry exclusion, the seam re-render and whether any test sets `DefaultEngine`.
