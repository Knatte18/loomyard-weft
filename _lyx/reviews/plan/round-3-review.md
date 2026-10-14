---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-shuttle-conditioned-send.md (TestRun_SendConditioned rows, the pending-record row over a pasteEngine)
    summary: The one row meant to cover the production paste path cannot reach it, since pasteEngine's session-signal parser drops APIERR, so the session never reads idle-stalled api-error and SendConditioned refuses with ErrSendConditionUnmet before pasting
    fix: In card 4, add the Edit target `internal/shuttleengine#clearableBoxEngine.ParseSessionSignals` (validated with validate-plan --overlay, no new blocking finding) and state in the Intent that the fake's parser gains the case `trimmed == "APIERR"` mapped to a signal of kind `SessionSignalAPIErrorTurnEnd`, so a pasteEngine, which embeds clearableBoxEngine through tuiEngine, reads an events file ending in APIERR as idle-stalled with cause api-error. Keep the pending-record row over the pasteEngine and have it assert the line went out through `PasteSubmit`, so the row pins that the paste path is taken.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-github-per-request-deadlines.md (Intent, the held-page row of TestListOpenIssues_PaginatesAndExcludesPullRequests)
    summary: The held-page row leaves page 1 only 30 ms of slack (a 170 ms handler under a 200 ms per-request budget, with the pause-record write and the marker write inside that budget), so a slow or -race run fails page 1 with a deadline error unrelated to the change the row pins
    fix: In card 3's row, use a 400 ms per-request budget, a first-page handler that answers after 250 ms (so a shared deadline has 150 ms left when page 2 starts), the same 150 ms gate Sleep, and a page-2 handler that answers after 250 ms. Reword the derived sentences to match, so page 1 has 150 ms of slack; with a fresh deadline page 2 has 400 ms after release, 150 ms of margin; under the shared deadline card 2's extension leaves page 2 only its 150 ms remainder after release, so it fails 100 ms short.
---

### [MEDIUM] Card 4's paste-path row cannot reach the paste path

**Location:** `_lyx/plan/04-shuttle-conditioned-send.md`, the last row of `TestRun_SendConditioned`.

**Issue:**
The row "pending record from an earlier send, over a `pasteEngine`" scripts the events file as `START` then `APIERR` and expects the line to be pasted, and the card says "The paste path is the one production's engine takes, so this row also covers it."
`pasteEngine` (`internal/shuttleengine/fakes_test.go:1095`) embeds `*tuiEngine`, which embeds `*clearableBoxEngine` (`internal/shuttleengine/run_test.go:1343`).
`clearableBoxEngine.ParseSessionSignals` recognises only `START`, `STOP:` and `SESSIONSTART`; an `APIERR` line yields no signal.
It also implements no `ActivityReader`, so the `STOP`-plus-activity route to cause `api-error` (`internal/shuttleengine/sessionstate.go:256`) is closed too.
The fold therefore reads the events file as an unmatched turn start, a busy session, and card 4's own `conditionedReading` refuses it with `ErrSendConditionUnmet` before typing anything.
The row fails as written, and every other row runs over `pickerEngine`, which is no `TextPaster`, so they all take the typing path.
Production's conditioned send goes through `typeVerified`'s paste branch (`pasteVerified`, `settleAndConfirmPasted`, `confirmPasted`), and as planned no reachable row pins it.
`sessionFakeEngine` (`fakes_test.go:805`) is where `APIERR` is parsed today; the paste-capable fake needs the same case.
Round 2's fixer added this row for its F3 and checked that `clearableBoxEngine` is a `SessionCycler`, which settles the paste branch, but not that its parser reads the API-error stall the row's condition needs.

### [MEDIUM] Card 3's held-page row leaves page 1 30 ms of slack

**Location:** `_lyx/plan/03-github-per-request-deadlines.md`, the held-page case of `TestListOpenIssues_PaginatesAndExcludesPullRequests`.

**Issue:**
The row runs `listOpenIssues` with a 200 ms per-request budget and a first-page handler that answers after 170 ms.
That leaves page 1 30 ms for everything else inside its deadline: the transport's `pauserecord.Network` call, which with a gate installed writes a marker file (`internal/pauserecord/hold.go:222`), the httptest round trip, the handler's own `pauserecord.Update` that makes the entry live, and decoding.
Under `-race` or a loaded gate slot that slack is thin, and a miss fails page 1 with a deadline error the row reads as a failure of the per-request change.
The other two margins are wide (page 2 fails 30 ms short under a shared deadline, and passes with 140 ms to spare under a fresh one), so the numbers can be rebalanced to give every margin at least 100 ms for under a second of wall time.
Round 2's F6 set these numbers to widen page 2's margin, and left page 1's slack at 30 ms, which that finding did not weigh.

The rest of the focus question is settled as not a defect:
`callTimeout` stays a const and the shortened budget is `listOpenIssues`'s `perRequest` parameter, so no test-shortened timeout is a package-level var (`PATTERN-test-speed`, "Time is injectable").
The row builds its client through `githubclient.NewWithBaseURL(server.URL, nil)`, whose `authRT.RoundTrip` calls `pauserecord.Network` (`internal/githubclient/transport.go`), and installs it through the `NewGitHubClient` seam, so page 2 reaches the gate; with `GH_TOKEN` set, `resolveToken` returns the env token before any cache read or `gh` shell-out.

## Focus verdicts with no finding

- Card 4, `pending-send.json` and `retryLanded`: the card keeps `retryLanded` in `sendVerified` and out of `typeVerified`, so `SendConditioned` never reads the record.
  `pasteVerified` still writes it on a paste that fails with `ErrSubmissionNotLanded` or `ErrSentTextInDraft`; a later plain `Send` of a different text clears it in `retryLanded` and reads it as not landed, and the next `revive` refuses on the line its earlier send left in the box, so the write is harmless.
- Card 5: every comment that says the orch session state decides nothing is on the card: the `Session.SessionState` comment (`internal/orchengine/watcher.go:42`), `logIdleStateDisagreement`'s clause (`watcher.go:603`), and both `doc.go` sentences (`doc.go:151`, `doc.go:154`).
  `orchcli/session.go:48` says only that the read uses the run's files, which stays true.
- Card 6, `restartSafe`'s branches through `Watcher.Run`:
  the record-read error is reached by the directory at `RecordPath`, since `pauseTick` logs an unreadable record and lets the tick succeed (`internal/orchengine/pause.go`), and `pauserecord.Read` fails on a directory;
  the state-read error is reached by the `onSend` row, since `deliverNotice` saves no state after a clean send (`heldTyping` returns at once on a nil error and `clearSendStuck` saves only under `StuckBySend`), so the tick succeeds and only the re-read fails, while `tick`'s leading `LoadState` fails the following ticks up to the consecutive-failure cap;
  the live hub entry is reached because `pauseTick` returns held with a nil error, so the tick succeeds and `restartSafe` then reads the entry.
