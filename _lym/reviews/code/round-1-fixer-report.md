---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: disputed
    ground: premise false
    rebuttal: "The branch cannot be reached through an unreadable anchor. hubWorkspaceFolders lists a pair only when lymcwd.ResolveWorktree resolves it, and that reads the anchor through the same anchorRelOrRoot call as lymcwd.NamedWorktree. A pair whose anchor is unreadable is skipped with a debug log, so it never becomes a beam entry and the warn branch never runs for it. The row the fix proposes, an unreadable beam anchor with that beam's file written without hook members, cannot produce its asserted state: the beam's file would not be written at all."
    evidence:
      - internal/fabricengine/hubworkspace.go:425
      - internal/fabricengine/hubworkspace.go:233
      - internal/lymcwd/lymcwd.go:127
      - internal/lymcwd/lymcwd.go:101
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

# Fixer-report, code round 1

Each fix is its own commit, once the gate was green (`lym gate test --upto 1` over `internal/boardserve`, `internal/boardengine`, `internal/boardcli` and `internal/vscode`: no failure).
F2, F5 and F6 share one commit, because they change the same hunks of `server.go` and `server_test.go`.
The self-check's three findings are in the last commit.

## Changes

- F1: `watchBoard` waits out a one-second floor (`watchEventFloor`, through `opts.After`) since its previous read on an event wake, then drains the queued events and reads once.
  The first event is read at once, because no earlier read exists.
  `TestRun` gained a burst row that counts one read per floor through the `Replaced` calls, and a row that pins the timer durations `1, 2, 4, 8, 8` seconds and the reset to one second on a change.
  `PATTERN-daemon-wakeups.md` names the floor.
- F2: the page sends no `after` on its first request only, and `waitForChange` answers at once only for an absent `after` key or one that differs from the revision.
  A `TestServer` row covers a board with no `board.json`.
- F3: `.gitattributes` pins `internal/boardengine/board.html` to LF.
- F4: `serveOptions` in `serveopen.go` builds the server's options, with the strand name cleared and the hub-owner registration.
  The new `serveopts_test.go` covers an orch strand name passing the refusal, the open deps' keep-warm cap for a named caller and none for an unnamed one, a board built by `serveOptions` accepting a task creation from a run agent's configured strand name, and `Register` recording the board server role under `runproc.HubOwner`.
- F5: only a request that passed the bearer check stamps the idle clock.
  `TransportRefusalsChangeNothing` runs on a clock that moves after construction and asserts each unauthenticated row leaves `lastAPIRequest` unchanged.
- F6: `checkRequest` refuses a create that carries a revision, with a `RefusedWritesChangeNothing` row.
- F7: `checkExpected` refuses a create of a project's slug with a plain error naming the project, with a `TestExpectingRevision` row.
- F8: `TestSetOwners` outcome rows assert the origin commit holds `board.json`, `README.md`, `board-data.js` and `board.html`; `TestSync_CommitsEveryFileTheRenderOfABoardWriteMade` asserts the same for the sync commit after a board write.
- F10: the revision assertion computes the hex SHA-256 of the stored entry's JSON in the test.
- F11, F12: `//testtiming:keep` lines above `TestViewerPage` and `TestKeybindingsBlock`.
- Gate repair: comments the gate flagged for a wrapped break or a line of 120 characters were rewritten as one sentence per line.
  They sit in `serveopts_test.go`, `sync_integration_test.go`, `doc.go`, `run.go`, `server.go` and `write.go`.
  A second pass shortened the doc line of `TestSync_CommitsEveryFileTheRenderOfABoardWriteMade`, which was still 120 characters; no code changed.
- Self-check: `doc.go` and the idle-bound comments of `run.go` now say the idle clock counts only authorized requests, `doc.go` states the floor and the no-`after` rule.

## Disputed

- F9, ground `premise false`: see the disposition entry above.
  A test row for the branch cannot be written, and removing the branch is not what the finding asks for.

## Deferred

None.

## Rulings

None.

## Self-check

- fix-consistency, LOW: the package doc and idle-bound comments still said "no API request" after F5 — fixed.
- fix-consistency, LOW: the package doc's lifecycle section omitted the event floor of F1 — fixed.
- fix-consistency, LOW: the package doc's long-poll contract omitted the no-`after` and empty-revision rules of F2 — fixed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
