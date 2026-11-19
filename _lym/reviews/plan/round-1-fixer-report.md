---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed wording claims neither a new connection nor a kept-alive one takes a request after the listener closes. net/http's closeIdleConns closes a StateNew connection only once it is about five seconds old, so a connection accepted just before the close still serves one request. Step 6 keeps the proposed order and calls but states that residual case, which the board's write lock keeps consistent."
  - id: F4
    disposition: ruling
    reason: "The parent recorded the decision (Added after Discussion, 2026-11-19: opener waited on through cmd.Wait, not proc.Reap) and ruled that card 6's startOpener bullet and the Shared Decisions bullet stand; both already say so, so no plan change was needed."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 1 fixer report

## Changes

- **F1** (card 4): `applyWrite` now says every op but `create` refuses an empty `revision` before any write, with the message `<op> of entry "<slug>" carries no revision, so nothing was written: way forward: reload the board and act on what it holds now`.
  The edit bullet's "no pre-check" became "no view pre-check".
  The Bound gains "Only `create` reaches the write section's empty-revision path, so no other op creates an entry."
  `TestServer`'s refused-and-changing-nothing list gains an edit with an empty `revision` of an absent slug.
- **F2** (cards 7 and 9): each Intent gains a `Bound:` paragraph worded as the fix proposed.
- **F3** (card 5, step 6), fixed differently: the step calls `SetKeepAlivesEnabled(false)` and closes the listener, then releases the lock, calls `beginDrain` and waits through `Shutdown`, ignoring the closed listener's error.
  It does not claim that no kept-alive connection takes a request.
  It states that the server takes no new connection, that each kept-alive connection closes after its current response, and that a connection accepted just before the close may still carry one request, which the board's write lock keeps consistent.
  The self-check's fix-proposal lens showed the proposed guarantee false (see Self-check).
- **F5** (card 6): `TestBoardServeScenario` gains a step after the second `open`: `lym board serve` from the prime's anchor exits 0 with `running: true` while the first server's state still answers `Probe`.
- **F6** (card 2): the `TestExpectingRevision` row for the revision of a following `Board.View` is cut.
- **F7** (cards 1 and 3): card 1 names `board-data.js` in `render.go`'s file header.
  Card 3 names `board.html` there, and both files in the comment over `replay.go`'s file-name constants.
  The self-check added the `# README` sentence in `doc.go` (see Self-check).
- **F8** (card 4): `**Extends:**` with `docs/overview.md`, an Intent sentence on the new `internal/boardserve/` package-tree line, and the proposed `**ImpactSummary:**`.
- **F9** (card 6): `internal/runproc/doc.go` moved from Extends to Edit, after `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`.
- **F10** (overview): the Shared Decisions revision bullet now says every op but a create refuses an empty revision first.
  It says an edit carries its revision straight to the write section, and a promote first refuses a task by its kind in the current `Board.View()`.
  The create sentence is kept.

`lym loom validate-plan` answers `ok: true` with informational findings only.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal, MEDIUM: card 5 step 6's claim that no kept-alive connection takes a request is false for a connection accepted under five seconds before the close.
  Fixed as proposed: step 6 is reworded with the lens's suggested wording.
- fix-affected-set, LOW: `internal/boardengine/doc.go:339`'s `# README` sentence on what every write renders goes stale.
  Fixed as proposed: card 3, which holds `doc.go` under Edit, now says that sentence also names `board-data.js` and `board.html`.

## Disputed

None.

## Deferred

None.

## Rulings

- **F4:** the parent recorded the decision with `lym loom decision add` ("Opener waited on through cmd.Wait, not proc.Reap").
  It ruled that card 6's `startOpener` bullet and the Shared Decisions bullet stand.
  Both already hold that text, so the plan is unchanged.
  The ruling is neither mistaken nor a redundant request for change, so I sent the parent no flag.
