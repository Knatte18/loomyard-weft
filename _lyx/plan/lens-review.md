# Lens review

All four forks returned a report.

- criteria-coverage, MEDIUM, D2 "a relaunch keeps its window": fixed — card 5's `ownWindowInsertArgs` takes the relaunch's old window and opens `-a -t` it, with a pure row and a tmux row for an `IfAbsent` relaunch.
- criteria-coverage, MEDIUM, D5 attach with a failed window-size or status readback: fixed — card 7's `TestAttachArgv_SelectsTheRelevantWindow` gains both rows.
- criteria-coverage, MEDIUM, no `SetContentRows` and no "not shown" line argued without a check: fixed — card 3 carries a `Verify:` grep that fails on either string in the two verbs' files.
- criteria-coverage, LOW, `statuspane/doc.go` import list keeps `internal/agentname`: fixed — card 3 drops it.
- criteria-coverage, LOW, D10 listing failure logs at Warn on every open: fixed — card 5 logs a failed listing at Debug; only a failed rename or move opens an episode.
- criteria-coverage, LOW, no-Selvage criterion not driven through attach or open: fixed — card 5's `TestEnsureShellWindow` drives that row through `OpenWindow` and `AttachArgv`.
- criteria-coverage, LOW, overview button first is unasserted: fixed — card 8's bar test asserts the `overview` button leads line 1.
- criteria-coverage LOW and forced-edits LOW (merged), `internal/orchcli/runs.go` header left stale: fixed — card 9 names it.
- forced-edits MEDIUM and row-reachability LOW (merged), shell window absent from `list-windows` would rename in every fake-driven test: fixed — card 5's `planShellWindowFix` plans nothing for an unlisted shell, pinned by a `TestEnsureShellWindow` row.
- forced-edits, LOW, two pane listings per applying re-apply break `listPanesPerReapply` and `waitForPromotion`: fixed — card 4 lists both and says what they account for.
- forced-edits, LOW, `StrandStatus.OwnWindow` meaning drifts: fixed — card 6 edits the field's doc.
- forced-edits, LOW, `watchPollWakeChangedPerOutcome`'s doc comment: fixed — card 4 adds the `lastFailed` rule to it.
- row-reachability MEDIUM and test-economy LOW (merged), accepted overview add launches with no fake: fixed — card 6 calls `validateDisplay` directly.
- row-reachability, MEDIUM, `OpenWindow` tmux rows need the sleeping `lyx` stub: fixed — card 5's `TestWindowOrder_RealTmux` injects it and runs non-parallel.
- row-reachability, LOW, the overview refusal row cannot check its text in the shared `rejected` table: fixed — card 6 makes it a subtest with its own seeded state and a substring check.
- row-reachability, LOW, the Info row needs the verbose log capture: fixed — card 5 names `logcapture.CaptureVerbose`.
- row-reachability, LOW, how card 8's hidden windows are made: fixed — plain `new-window` plus `@lyx_hidden`, as `newBarFixture` does.
- row-reachability, LOW, `handleWatchOutcome` on a skip at the failed box: fixed — card 4 states a skip at `lastFailed` leaves both boxes as they were.
- test-economy, MEDIUM, `relevantWindow` tested at four sites: fixed — card 7 drops the pure rows and keeps the newest-live-agent removal in `TestOwnWindow_RealTmux` only.
- test-economy, MEDIUM, list-clients failure row duplicated: fixed — card 7 keeps it in `TestAttachArgv_MultiClientWarning` only.
- test-economy, MEDIUM, shell check tested at planner and engine level: fixed — card 5 drops the `planShellWindowFix` pure rows and keeps `TestEnsureShellWindow`.
- test-economy, LOW, `AttachArgv(0, 0)` shell fix tested twice: fixed — card 5 drops the tmux repeat.
- test-economy, LOW, "waits on nothing" row repeats an existing row: fixed — card 2 drops it and states the existing row pins it.
