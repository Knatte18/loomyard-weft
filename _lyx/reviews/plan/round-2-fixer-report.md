# Fixer report — plan, round 2

## Changes

- **F1** (fixed as proposed), `03-github-per-request-deadlines.md`: `callTimeout` stays a const, with its comment change.
  Added the Create target `plan:internal/selfreportengine#listOpenIssues` -> `func listOpenIssues(perRequest time.Duration) ([]Issue, error)`.
  `ListOpenIssues` returns `listOpenIssues(callTimeout)`, whose page loop derives a `perRequest` deadline per page.
  The held-page row calls `listOpenIssues` with its budget; the package-`var` sentence and the set-and-restore clause are gone.
- **F2** (fixed as proposed), `03-github-per-request-deadlines.md`: the held-page row builds its client with `githubclient.NewWithBaseURL(server.URL, nil)` and installs it through the `NewGitHubClient` seam in place of `installGitHubClient`.
  It redirects the credential cache and sets `GH_TOKEN`/`GITHUB_TOKEN` as `TestAuthRT_WaitsWhileAPauseCoversTheProcess` does, and asserts the gate's `Sleep` ran once.
- **F3** (fixed as proposed), `04-shuttle-conditioned-send.md`: `retryLanded` stays out of `typeVerified`; `sendVerified` is `awaitIdleSession`, then `retryLanded` on the paste path, then `typeVerified`, so `Runner.SendConditioned` never consults `pending-send.json`.
  Added a `TestRunner_SendConditioned` row over a `pasteEngine` with a pending record for the same text; the line is pasted.
  `pasteEngine` embeds `tuiEngine` over `clearableBoxEngine`, a `SessionCycler`, so the row takes the paste path.
- **F4** (fixed as proposed), `05-orch-conditional-notice.md`: added `internal/orchengine#Watcher.logIdleStateDisagreement` to the Edit list after `Session`.
  The Intent now rewrites its doc comment's clause and `doc.go`'s session-state paragraph to name the conditioned notice's drop as the one decision that reads the state.
- **F5** (fixed with the reaching-row option), `06-orch-watch-binary-reexec.md`: added a row whose `fakeSession.onSend` hook replaces the state file with a directory while the idle tick delivers a queued notice.
  After `Send`, `deliverNotice` calls `heldTyping` (no save on a nil error), `clearSendStuck` (saves only when `StuckBySend`) and `RemoveNotice`, so the tick succeeds and only `restartSafe`'s re-read fails.
  The check never runs, and `Run` returns the `LoadState` error at its consecutive-failure cap.
  I did not drop the branch, because then `tick` would have to return the phase it ended on from each of its return sites.
- **F6** (fixed as proposed), `03-github-per-request-deadlines.md`: the budget is now 200 ms per request, page 1 answers after 170 ms and page 2 after 60 ms.
  That leaves 30 ms under a shared deadline and 140 ms of margin under a fresh one.

## Disputed

None.

## Deferred

None.
