---
verdict: APPROVED
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/orch-continue/status.json` names `main` as `product.parent`, and no `_lyx/loom/rework/` round exists.
The four commits implement cards 1 to 4, and I found no defect.

What I checked against the cards:

- Card 1: `newestSessionID` holds the pick `readSessionFacts` made before, and `readSessionFacts` calls it for liveness.
  `readRunSessionState` sets the id on the parser path and falls back to `rs.SessionID` on the no-parser path.
  `Run.SessionID` exists, and `TestReadSessionStates` and `TestRunner_Start_HappyPath` carry the card's rows.
  No caller outside `internal/shuttleengine` changed, and none needed to.
- Card 2: `State.SessionID` is `session_id`, and `ResetForFreshLaunch` keeps it.
  `recordSessionID` logs a read error at Debug and returns nil, writes nothing for an empty or unchanged id, and saves through the strand-checked `w.save`.
  It runs once per tick after the strand-alive check and `initCursor` and before `ReadEvents`.
  The test-budget bump of one matches the one new top-level test, and the rebound-strand and the four-row tests match the card.
- Card 3: the `--continue` guards sit where the card places them.
  The exclusivity checks run ahead of the lock, and the live and no-id checks run before `RemoveStrand`.
  `--continue` takes the adopt branch through `resumeSessionID`.
  Every fresh launch saves `started.SessionID`, and `previousSessionHint` is read from the state loaded before the save.
  The watcher-stopping hint is composed with it in the card's order, `adoptStrand` clears the id on a rebind, and `session_id` is added to the envelope under `--continue`.
  `StartSession` has no caller outside `internal/orchcli`, and the adapter and both doubles are re-signed.
  The `Refusals`, `StarterError`, `LiveStrand` and `AdoptLaunchesResumeSpec` tables carry the card's rows as table rows, so there are no new top-level tests and no duplicate coverage.
- Card 4: `statusFields` and the `Long` text carry `session_id`, `stop` is untouched, and the "continue after stop" row pins that it keeps the id.
- Docs: the `doc.go` changes in `internal/orchcli` and `internal/orchengine` match the built behavior, including the new `Continuing after a reboot` section and the open-risk and residual entries.
- Mechanical checks: `lyx gate test` over `internal/orchcli`, `internal/orchengine`, `internal/shuttleengine` and `cmd/lyx` is green, and `lyx loom lint-comments` reports ok.
