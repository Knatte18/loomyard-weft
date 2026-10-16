# Plan round 2 — fixer report

## Changes

- **F1** (card 3), fixed as proposed.
  `callTimeout` stays a const, and its comment change stays.
  A new Create target `plan:internal/selfreportengine#listOpenIssues` -> `func listOpenIssues(perRequest time.Duration) ([]Issue, error)` follows the Edit list.
  `ListOpenIssues` returns `listOpenIssues(callTimeout)`, whose page loop derives a fresh `perRequest` deadline per page.
  The held-page row calls `listOpenIssues` with its short budget.
  The "becomes a package `var`" sentence and the "sets `callTimeout` ... and restores it" clause are gone.
- **F2** (card 3), fixed as proposed.
  The held-page row builds its client with `githubclient.NewWithBaseURL(server.URL, nil)` under `t.Setenv("GH_TOKEN", ...)` and `t.Setenv("GITHUB_TOKEN", "")`, installs it through the `NewGitHubClient` seam in place of `installGitHubClient`, and asserts the gate's `Sleep` ran once.
- **F6** (card 3), fixed as proposed.
  The budget is now 200 ms, with page 1 answered after 170 ms and page 2 after 60 ms: 30 ms left under a shared deadline, 140 ms of margin under a fresh one.
  The row's description now reads "held by a pause past what a shared deadline has left", because the 150 ms hold no longer outlasts the whole budget.
- **F3** (card 4), fixed differently in one detail.
  `typeVerified` leaves out the `retryLanded` check.
  `sendVerified` is now `awaitIdleSession`, then `retryLanded` on the paste path, then `typeVerified`, so `Runner.SendConditioned` never consults `pending-send.json`.
  The new `TestRunner_SendConditioned` row covers the pending send on the paste path.
  The fix said to run that row over a `pasteEngine`; I did not, because `internal/shuttleengine/fakes_test.go`'s `pasteEngine` wraps `tuiEngine` over `fakeEngine`.
  That base parses no session signals and is no `SessionCycler`, and `sendVerified` takes the paste path only for a `TextPaster` that is also an `InputBoxReader` and a `SessionCycler`.
  The row instead runs over a `pastePickerEngine`, declared in `conditionedsend_test.go`: a `pickerEngine` that also implements `TextPaster` with `PasteSubmits` true.
  This matches `claudeengine`'s `pasteSubmitsOnEnter`.
  The row's turn start comes from the clock's sleep hook, as in the clean row.
- **F4** (card 5), fixed as proposed.
  `internal/orchengine#Watcher.logIdleStateDisagreement` is now on the Edit list after `internal/orchengine#Session`.
  The Intent now says this: that comment's clause and `doc.go`'s session-state paragraph name the conditioned notice's drop as the one decision that reads the state.
- **F5** (card 6), fixed with the reaching row the fix offered.
  The new row: the `fakeSession`'s `onAlive` hook replaces the state file with a directory on the second tick only, and the sleep after that tick restores the file.
  So the check runs on every tick but that one.
  `restartSafe`'s re-read and its error branch stay.
  I did not take the drop option, because the phase a tick ran on is not the phase it ends at: `tickIdle` can leave idle within that tick, so the drop would make `restartSafe` report safe for a tick that started a cycle.
  The row's premise was checked in `internal/orchengine/watcher.go`.
  From the second tick on, a quiet idle tick saves nothing after `StrandAlive`: `tickIdle` returns before any save when no trigger holds, a repeated hold reason does not save again in `logHeld`, and `colorPending` is cleared or held without a save.

- **Gate `edit-test-file/12-tender-rename`** (card 12), fixed.
  The card listed `cmd/lyx/seamsignature_test.go` under Edit as a whole file, but that file declares only two blank `var _` slices, and `lyx quarry toc` lists no symbols for it, so it had no member glyph to name.
  Card 12 now names the two slices `runCLISeams` and `runCLIInSeams`, declared as Create targets `plan:cmd/lyx#runCLISeams` and `plan:cmd/lyx#runCLIInSeams`, and drops the file path from Edit.
  The Intent says the elements stay unchanged, each comment says "The var below" in place of "The blank identifier below", and the compile stays the assertion.
  The "pure rename" sentence now notes this one exception.

`lyx loom validate-plan` returns `"ok":true`, and its only findings are informational.

## Disputed

None.

## Deferred

None.
