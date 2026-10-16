# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING), fixed as proposed: in card 3, `callTimeout` stays a const and only its comment changes.
  The card gains the Create target `plan:internal/selfreportengine#listOpenIssues` -> `func listOpenIssues(perRequest time.Duration) ([]Issue, error)`.
  `ListOpenIssues` returns `listOpenIssues(callTimeout)`, and the page loop derives `context.WithTimeout(context.Background(), perRequest)` once per page.
  The held-page row calls `listOpenIssues` with its short budget.
  The package-`var` sentence and the set-and-restore clause are gone.
  The ImpactSummary now says "no exported signature changes", because the new unexported function exists.
- **F2** (MEDIUM), fixed as proposed: card 3's held-page row sets `GH_TOKEN` and clears `GITHUB_TOKEN`, builds its client with `githubclient.NewWithBaseURL(server.URL, nil)` and installs it through the `NewGitHubClient` seam in place of `installGitHubClient`.
  It also asserts that the gate's `Sleep` ran once.
- **F3** (MEDIUM), fixed differently.
  The `retryLanded` move is as proposed.
  `sendVerified` is `awaitIdleSession`, then `retryLanded` on the paste path, then `typeVerified`.
  `typeVerified` holds only the paste and typing paths, and `SendConditioned` never consults `pending-send.json`.
  The difference is the new row's engine.
  The review proposed a `pasteEngine`, which I did not use: it is built on `tuiEngine` and `clearableBoxEngine`, and `clearableBoxEngine.ParseSessionSignals` (`internal/shuttleengine/run_test.go:1373`) has no `APIERR` case.
  Under it the run could never read idle-stalled `api-error`, so `conditionedReading` would refuse before any paste.
  The row instead runs over `pickerEngine` plus a `TextPaster` pair (`PasteSubmits`, `OwnPastePlaceholder`) declared in `conditionedsend_test.go`.
  `pickerEngine` already parses `APIERR` through `sessionFakeEngine` and is a `SessionCycler` and an `InputBoxReader`, so the send takes the paste path.
  The Bound paragraph and the `doc.go` paragraph also list the dropped pending-send retry.
- **F4** (LOW), fixed as proposed: card 5's Edit list gains `internal/orchengine#Watcher.logIdleStateDisagreement` after `Session`.
  The Intent says that its doc-comment clause, and `doc.go`'s two session-state sentences, now name the conditioned notice's drop as the one decision that reads the state.
- **F5** (LOW), fixed by the review's first option, a reaching row.
  Card 6 gains a row for the state turning unreadable after a successful tick.
  On the second tick, the `fakeSession`'s `onAlive` hook moves the state file aside and puts a directory in its place.
  An idle tick with no events, notices or trigger saves nothing afterwards, so `restartSafe`'s re-read fails and the check runs on ticks 1 and 3 only.
  The `sleep` closure restores the file.
  The review counted the record half as already reached, but card 6 named no row for it, so the card also gains a row with a directory at `RecordPath`.
  I traced both rows against `Watcher.tick`, `initCursor`, `typeStartColor`, `deliverNotice` and `pauseTick` (`internal/orchengine/pause.go:48-54`).
- **F6** (LOW), fixed as proposed: card 3's budgets are now 200 ms per request, page 1 answered after 170 ms and page 2 after 60 ms.
  A shared deadline leaves page 2 30 ms, and a fresh one leaves 140 ms of margin.

## Gate repair

- **edit-test-file/12-tender-rename** (blocking), fixed differently.
  The gate asks for the changed members of `cmd/lyx/seamsignature_test.go` to be named as glyphs under Edit, which I could not do.
  The file declares only two blank-identifier slices (`var _ = []func(...)`), and `lyx quarry toc cmd/lyx/seamsignature_test.go` reports `"symbols": []`, so there is nothing to name.
  The file path is now off card 12's Edit list.
  The Intent names the file by path, says it holds no member glyph, and states its edit: the `internal/tendercli` import and the `tendercli.RunCLI` and `tendercli.RunCLIIn` slice entries, made as part of the rename's import updates.
  `lyx loom validate-plan` now returns `ok: true`, with only the create-new-unit informational findings.
  The gate's create-new-unit findings are informational and need no change.

## Self-check

- consistency, MEDIUM, fixed differently: after F6's timings, card 3's row opening ("held by a pause past the per-request budget") no longer held, since the 150 ms hold is shorter than the 200 ms budget.
  The fork offered two changes, and I did not raise `Sleep`, because the timings are F6's and the row already separates the two cases.
  I reworded the opening to "held by a pause past what a deadline shared by both pages has left".
- claims: no findings.
- way-forward: no findings.

## Disputed

None.

## Deferred

None.
