Task: check the fix-consistency lens over the change set (twins and siblings of the changed text).

Findings:

1. **LOW**: `internal/shuttleengine/await.go` (`Runner.AwaitForks`, error text "find the run of strand %q") now differs from its twins.
   - Evidence: `internal/shuttleengine/session.go` lines 41, 65, 93, 150, 173, 194, 216, 303 and 349 all still wrap a `FindRun` error as `"shuttle: %q is not a shuttle strand: %w"`, and `enginedispatch_test.go:631` asserts that text.
   - The F6 wording is more accurate. It leaves `AwaitForks` as the only caller phrasing the failure differently.
   - This follows the review's own fix key, so I count it only as a noticeable sibling mismatch.

2. **LOW**: The `await-forks` verb text is not updated for the new bound. `internal/shuttlecli/awaitforks.go` lines 22 and 34-35 say `ready false` means "telling the caller to run the command again", with no limit. The three stencils now say to stop after five `ready: false` answers in a row.
   - A reader of `--help` sees an unbounded re-run instruction, and the stencil, which is the authority for roles, says otherwise.
   - The verb's contract can stay unbounded, but the help text should not contradict the stencils.

3. **LOW**: The `background.go` file header was not updated for the changed behaviour.
   - Lines 7-8 say a launch is outstanding only when no notification names "the task id the launch's tool result returned".
   - Line 10 says the same transcript reading answers which forks are running.
   - `unfinishedForks` now also counts a fork launch with no tool result yet, with an empty ID. The header never says so, and the header is the file's contract.
   - The `OutstandingForks` doc comment also omits it. The `OutstandingForks` doc does say an unreadable transcript is an error, and that part still holds.

Checked, no inconsistency:
- The three stencils carry identical wording ("up to five answers in a row", "a sixth time in a row").
- `strandResolver` is satisfied by `*reedengine.Engine` at the `cli.go` assignment.
- `transcriptHasUnmatchedLaunch` keeps its old behaviour through the `countPending=false` wrapper, and the Stop path at `background.go:102` still uses it.
- The `docs/overview.md`, `burlerengine/doc.go` and `twilldirective/doc.go` mentions of `await-forks` contain no re-run or bound wording.

Tool calls: 1 (a grep over the repo for re-run wording and the "not a shuttle strand" text), because my context lacked the sibling occurrences outside the changed files.
