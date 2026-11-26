```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails if it breaks.

**Per behaviour, the test that pins it**

- **`readClientFormatsVia`, the client's own row among others, including a `\r\n` line ending.** `TestSwitchClientVia_IssuesOnlyTheThreeCommands/StepsFromTheClientsOwnSessionAmongOtherClients` and `TestClientAgentPaneVia/OtherClientsRowsAreSkipped` (`internal/reedengine/remotecontrol_test.go:36`).
- **`readClientFormatsVia`, a failed `list-clients` call.** `ClientListingFails` (`switch_test.go`) and `TestClientAgentPaneVia/TmuxError` (`remotecontrol_test.go:40`).
- **`readClientFormatsVia`, a client that is not attached.** `UnknownClient` (`switch_test.go`, exact text `no client "/dev/pts/3" is attached`), plus `ClientNotListed` and `EmptyOutput` (`remotecontrol_test.go:41,44`).
- **`readClientFormatsVia`, a row with the wrong field count.** `TestClientAgentPaneVia/TooFewFields` (`remotecontrol_test.go:42`).
- **`readClientFormatsVia`, an empty trailing format.** `PaneWithNoStrandName` and `AgentPaneWithoutTheMark` (`remotecontrol_test.go:33-34`). `TestRunButtonVia/StrandEntryWithAnEmptyReadRunsNothing` feeds the row `c1\t`.
- **`readClientFormatsVia`, the argv `list-clients -F #{client_name}\t…`.** `wantRead` in the switch test and `wantArgv` in `TestClientAgentPaneVia:24`.
- **`switchClientVia` steps from the client's own session.** Tier 0: the switch test rows. Tier 2: `TestSwitchClient_StepsFromTheClientsOwnSession`, which would fail on the old read because the "third" client attaches last.
- **`switchClientVia` with the current session missing from the listing.** `CurrentSessionMissingFromListing`.
- **`buttonStrand` reads `#{@lym_strand_name}` from the clicking client's row.** `TestRunButtonVia` rows such as `StrandEntryRunsWithTheClickedPanesNameQuoted`, whose fake puts another client's row first (`buttonclick_test.go:169`). Tier 2: `TestRunButton_StrandNamesTheClickingClientsOwnPane`, which would fail on the old read because the "other" client attaches last.
- **`buttonStrand` with an empty read or a failed read.** `StrandEntryWithAnEmptyReadRunsNothing` and `StrandEntryWithAFailedReadRunsNothing` (`buttonclick_test.go:101-102`).
- **`buttonStrand` when the clicking client has no row.** No row. The behaviour shares the failed-read branch, and the helper's not-attached path is pinned by the switch and RC rows. No finding.
- **`clientAgentPaneVia` moved onto the helper, empty pane id.** `EmptyPaneID` (`remotecontrol_test.go:43`). The other rows stay green.
- **`session-bus` row denies and allows.** `TestRows_PatternsDenyAndAllow` rows `systemd_run_user_scope` through `inside_a_double_quoted_argument`, and `x; systemd-run --user true`.
- **`hub-tmux-server` row denies and allows.** The `named_server` through `flag_of_a_later_command` rows, and the `reedengine.ServerName` drift check (`rows_integration_test.go:727-731`).
- **Both rows installed once in every claude session, and their notices announced.** `TestBuildSettings_NoAgentHookAndForkGuard` (one each), `TestBuildSettings_DenyToggleMatrix` (count 10), `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` (10 and 11) and `TestBuildDenyNotice_SentenceContent`.
- **Codex renders both rows.** No diff test. The codex engine derives its rows from the table, so the codex tests that render every row cover this. Not a finding of this lens.
- **Loosened checks.** None. `TestRunButtonVia` dropped its `-p` filter, so every `display-message` call must now be the four-argument message form, which is stricter. The `UnknownClient` `wantErr` is more specific. The raised counts follow the two new rows.

**Findings**

1. **LOW, design (test gap).**
   - Location: `internal/shellguard/rows_integration_test.go:687-714`.
   - Summary: neither new row has a test row for a command inside a double-quoted `sh -c "…"` script. That is the third scope `commandRowPattern` renders, and it reads arguments its own way: `scope.argument()` and the `(\\"|')?` quote-opening in `hubTmuxServerCommandPattern`.
   - Sibling rows cover that scope: `rows_integration_test.go:122,138,149,199,222`.
   - So a mistake in either new pattern's double-quoted alternative would pass every test.
   - Fix: add one deny row per new row, such as ``bash -c "busctl --user list"`` and ``sh -c "tmux -L lym-hub-1a2b3c4d ls"``.
   - Evidence: `rows.go:596-598`, `rows.go:618-623`, `rows.go:217-240`.

2. **LOW, design (test gap).**
   - Location: `internal/shellguard/rows_integration_test.go:703-714`.
   - Summary: no row covers the single-quote opening `(\\"|')?` that `hubTmuxServerCommandPattern` accepts after `-L` or `-S`, such as `tmux -L 'lym-hub-1a2b3c4d' ls` or `tmux -S '/tmp/tmux-1000/lym-hub-x' ls`. Only a double-quoted `-S` path is tested.
   - Fix: add one deny row for a single-quoted `-L` name.
   - Evidence: `rows.go:616,621`.

Outside my lens, one sentence: the summary says the RC click's errors are unchanged, but the not-attached and unreadable-row errors now arrive wrapped in `reed: read the active pane of client …: …`. `TestClientAgentPaneVia` matches by `Contains`, so it does not notice.

Tool calls:
1. `TestClientAgentPaneVia` and the `TestRunButtonVia` row list: the manifest had the diff hunks but not the unchanged test rows that pin the helper's error paths.
2. A grep for double-quoted `sh -c` rows that matched nothing: a quoting attempt to see whether sibling rows test that scope.
3. The same grep with corrected quoting, which found the sibling rows.
