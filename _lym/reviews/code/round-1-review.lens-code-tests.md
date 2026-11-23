```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
1. **A pre-run refusal of `toggle` (the directory is no worktree) redraws the clicked bar with `lym: <refusal>`.** Pinned by `TestToggle/ARefusalBeforeTheToggleStartsIsShownOnTheBar` (`internal/slackcli/toggle_test.go:173-186`). It asserts a literal redraw and fails on the merge base, which makes no redraw.
2. **A pre-run refusal from a failed wiring is shown on the bar.** The branch is the same as in 1, so it is covered through 1. No row of its own; that is acceptable.
3. **A pre-run refusal of any other verb draws nothing on the bar (the `cmd.Name() == toggleVerb` guard).** None. `PreRunResolvesNoWorktreeForTheRelayAndBarButDoesForEveryOtherVerb` (`internal/slackcli/cli_test.go`, around line 1124) refuses `status` under the pre-run but never asserts `h.redraws` is empty.
4. **The verb body still redraws through `reportToggle`, reading the flags from `cmd.Flags()`.** Pinned by the `TestToggle` rows (`wantRedraw("")`, `toggle_test.go:167`) and the `TestToggleSlackRefusals` rows (`toggle_test.go:250`). They check the tmux path, socket and client from the flags, and the message.
5. **Every failed toggle is logged at Warn** (`toggle.go:156`). None. No test captures the logger.
6. **The toggle's message is held for 10000 ms with `display-message -d`.** Pinned by `TestShowToggleOutcome` rows `MessageIsShownThenRedrawn` and `HashAndPercentInTheMessageAreDoubled` (`internal/reedengine/slacktoggle_test.go:65-70`), which use literal `"-d", "10000"`.
7. **`showToggle` is set in `productionCLI`, before wiring, so a production toggle whose wiring failed still reaches `ShowToggleOutcome`.** None. The harness supplies its own `showToggle` (`cli_test.go:327`), and the reach report shows `wireProduction` unreached. No check would fail if the assignment moved back into `wireProduction`, where a pre-run failure would call a nil func.

**Findings:**

- **MEDIUM**, class `design`, `internal/slackcli/toggle.go:156`: the Warn trace-log line for a failed click has no test.
  - The board entry (issue #212) asks that "the bar or a log shows why it did not". The bar half is pinned; the log half is not, so removing the line would bring back half the bug with every test green.
  - `internal/testkit/logcapture` exists and other packages use it (for example `internal/websterengine/recoverbatch_test.go`). Adding an assertion that the Warn record exists, with `toggle` and `err`, to the pre-run row and one refusal row would pin it.
  - The change description defends the gap as "for a person reading the trace" (`_lym/landing/summary.md:33`). But the goal names this line as the deliverable, so it is not mere human-read wording.
- **LOW**, class `design`, `internal/slackcli/wiring.go:56`: nothing checks that the production CLI sets `showToggle` before wiring.
  - Without that check, a move back into `wireProduction` (`wiring.go:136` on the base) would nil-panic on every pre-run-failed click in production, and the tests would not notice. Evidence: harness `cli_test.go:327`; the reach report says "no mapped test reaches internal/slackcli.wireProduction".
  - The fix is an assertion near the pre-run test that `productionCLI().showToggle` is non-nil, or a run of `Command()` with a failing cwd and a fake tmux.
- **LOW**, class `design`, `internal/slackcli/cli.go:196`: no row checks that a pre-run refusal of a verb other than `toggle` draws nothing.
  - Without the verb-name guard, a refused `status` would call `showToggle` with empty flags, and `PreRunResolvesNoWorktreeForTheRelayAndBarButDoesForEveryOtherVerb` (`cli_test.go`, around line 1137, the `status` step) would still pass.
  - The fix is to assert `len(h.redraws) == 0` after that `status` step.

Tool calls:
- `ls internal/testkit | grep -i log` plus a grep for users of `logcapture`: to confirm a log-capture kit exists before proposing a Warn assertion. The manifest did not list the testkit packages.
