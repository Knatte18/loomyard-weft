# Card 5 — `lyx reed follow`: the jump verb

**Create:**
- `internal/reedengine#overviewWindowOption`
- `internal/reedengine#followTargetFormat`
- `internal/reedengine#followTargetWindow`
- `internal/reedengine#FollowSession`
- `internal/reedengine#followSessionVia`
- `internal/reedengine#TestFollowSessionVia`
- `internal/reedcli#reedCLI.followCmd`
- `internal/reedcli#validateFollowFlags`
- `cmd/lyx#reedSubcommand`

**Edit:**
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#TestFollow_RealTmux`
- `internal/reedengine#TestWindowOrder_RealTmux`
- `internal/reedengine/doc.go`
- `internal/reedcli#Command`
- `internal/reedcli#TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository`
- `internal/reedcli#TestRunCLI_ColdHub`
- `internal/reedcli/doc.go`
- `cmd/lyx#TestSkipStencilSeed_HonoursTheAnnotation`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `cmd/lyx#reedSwitchCommand`

**Uses:**
- `internal/reedengine#followSessionOption`
- `internal/reedengine#relevantWindow`
- `internal/reedengine#SwitchClient`
- `internal/reedcli#reedCLI.switchCmd`

**Intent:**
D8: a verb told its geometry that turns follow on and selects the session's relevant window from tmux facts alone, for the follow button (card 6) to run.

`openOwnWindowLocked` sets the window option `@lyx_overview` (`overviewWindowOption`) to `1` on an overview window beside `@lyx_own`, logged and never fatal like the other window options, so a told-geometry caller can tell the overview from an agent window without reed state.

In `follow.go`:
`FollowSession` builds `newTmuxCmdForSocketPath(tmuxPath, socketPath)` and calls `followSessionVia`, as `SwitchClient` does; it reads no working directory, config or state.
`followSessionVia` issues, in order and nothing else: `list-panes -s -t <exactSessionTarget(session)> -F followTargetFormat`; `set-option -t <exactSessionTarget(session)> @lyx_follow on`; and, when `followTargetWindow` of the listing is non-empty, `select-window -t <that window>`.
A failed listing, which is how an unknown session answers, returns an error naming the session before any write; a failed set-option returns an error before any select; a failed select returns an error naming the window.
`followTargetWindow` is `relevantWindow`'s rule over tmux facts: among windows carrying `@lyx_own` and not `@lyx_overview`, the highest-index one holding a pane that is not dead, else the window carrying `@lyx_overview`, else the empty string; a line it cannot parse is skipped.
Its doc comment says it is the same rule as `relevantWindow`, read from window options instead of reed state, and that an overview window opened by an older lyx carries no marker and so counts as an agent window, which it can win only when no other agent is live, the same outcome as the overview fallback.
The bound: the verb sets one session option and selects at most one window of the told session, on the told socket.

`TestFollowSessionVia`, in a new file `internal/reedengine/follow_test.go`, drives `followSessionVia` through `installFakeTmuxOn` on a `TmuxCmd`, asserting the exact call sequence per row:
the newest live agent is selected over an older live agent, a dead newer agent, the overview and a non-own window; with no live agent the overview is selected; with neither, `set-option` runs and no `select-window`; an unmarked own window (an overview opened by an older lyx) with a live pane is selected when no other agent is live; a listing line that does not parse is skipped and the rest still decides; a failing `list-panes` returns an error naming the session and issues no `set-option`; a failing `set-option` returns an error and issues no `select-window`; a failing `select-window` returns an error naming the window, after the `set-option`; and a session name holding a space and a `#` reaches both targets as `=<name>` unchanged.
`TestFollow_RealTmux` (card 3) gains subtests: `FollowSessionTurnsFollowOnAndSelectsTheNewestAgent` (follow set off, viewer on the shell, `FollowSession` with the server's `#{socket_path}` and the engine's tmux path reads back `on` and lands the viewer on the newest live agent window); `FollowSessionWithOnlyTheOverviewSelectsIt` (every agent strand removed, follow off and the viewer on the shell, it reads back `on` and lands the viewer on the overview window, found through the real `@lyx_overview` marker); `FollowSessionWithNoAgentOrOverviewSelectsNothing` (the overview strand removed too, it reads back `on` and the viewer stays on the shell); and `FollowSessionOnAnUnknownSessionFailsAndChangesNothing` (a session name the server does not hold returns an error and leaves the real session's `@lyx_follow` and current window unchanged).
`TestWindowOrder_RealTmux`'s `AnOverviewAddedAfterAgentWindowsHoldsTheLowestIndexAndCarriesNoMarks` reads `#{@lyx_overview}` beside its other marks and wants `1` on the overview window, and none of its agent windows carries it.

In `internal/reedcli`, a new file `follow.go` holds `followCmd`, `Use: "follow"`, modelled on `switchCmd`: flags `--socket`, `--session` and `--tmux`, all required and checked by `validateFollowFlags`, which names the missing flag; annotations `clihelp.AudienceRole` and `clihelp.SkipStencilSeedAnnotation`; `cobra.NoArgs`; on success it prints nothing, and a refusal or a `FollowSession` error is printed as an `output.Err` envelope with a non-zero exit.
Its long help says it reads no working directory, config or state and is run by the follow button through run-shell.
`Command` registers it beside `switchCmd`.
`TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository` gains rows `follow missing session`, `follow missing socket` and `follow missing tmux`, each naming its flag, from the same non-repository directory, and its doc comment names both verbs.
`TestRunCLI_ColdHub` gains a subtest, run once its session is up, that calls `RunCLIIn` with `follow` told the server's `#{socket_path}`, the session's name and the absolute tmux path: it exits 0 with empty output; told a session name the server does not hold, it exits 1 with an error envelope naming that session.
In `cmd/lyx`, `reedSubcommand` replaces `reedSwitchCommand`, finding the reed subcommand by the name it is told, and `TestSkipStencilSeed_HonoursTheAnnotation` gains the row `reed follow carries the annotation` beside its switch row, which now calls `reedSubcommand(t, "switch")`.

`test-budget.yaml`'s `internal/reedengine` row rises by one for `TestFollowSessionVia`.
`internal/reedengine/doc.go` gains a paragraph beside `SwitchClient`'s on `FollowSession` and the `@lyx_overview` marker; `internal/reedcli/doc.go` gains one on `lyx reed follow`.

**ImpactSummary:** New engine function and CLI verb with no existing callers; `openOwnWindowLocked` gains one non-fatal window option on overview windows; `Command` registers one more subcommand; `reedSwitchCommand`'s one caller moves to `reedSubcommand`; existing CLI, stencil-gate and real-tmux tests gain rows, and the budget row moves by one.

**Commit:** 5: reed: lyx reed follow turns follow on and selects the relevant window
