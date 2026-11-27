---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: ruling
    reason: "The parent's added decision \"A first card runs the KWin probes\" settles F4; it was carried out by inserting card 1, kwin-probes, and moving every recording and operator step out of cards 6 and 8."
  - id: F5
    disposition: deferred
    reason: "The reader analysis is in card 12. The second half asks either to limit registration to an opening verb inside a run or for a parent decision that every non-prime child registers. Limiting would leave a beam's teardown unable to reach its child with the SIGTERM D12 relies on, and lym has no signal that tells an opening verb it runs inside a run, so the remaining choice is the parent's lym loom decision add, which a fixer cannot record."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "Cards 8 and 12 name their new test files as proposed. Card 13's test goes into a new tier1 file, internal/reedcli/handterminalwindow_integration_test.go, not into placewindow_integration_test.go, whose file comment says it runs placeTerminalWindow only and would go stale; that comment is a _test.go header, which edit-test-file keeps a card from targeting by file."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Plan fixer-report, round 1

`lym loom insert-card 1 kwin-probes` renumbered every card by one, so the review's cards 5 to 12 are now cards 6 to 13.
Card numbers below are the new ones.
`lym loom validate-plan` answers `ok: true` with informational findings only.

## Changes

- **F1** (card 11): mode `off` no longer records in step 3.
  It skips the parent wait, takes `kwinplace.lock` from the shared budget, runs the locked `Query`, records `reportUpdateFrom` over that snapshot alone, releases the lock and runs `afterPlace`.
  The `off` row now asserts that the record follows the locked query.
  A new row has another flow's apply land between the off child's window-wait query and its record, and asserts the placed window's home is kept.
- **F2** (cards 10, 11): `PlanInput` gains `Prime`.
  A homeless prime window starts at the anchor under both `same` and `next`.
  `Place` passes `req.Prime`, and new `TestPlan` and `TestFlow_Place` assertions check that the prime under `next` lands on the current desktop.
- **F3** (cards 6, 10, 11): card 10 exports `CurrentOrder`, D6's current-order rule, which `Plan` and the new card 11 `reportUpdateFrom` both call.
  `TestPlan`'s rows that pinned that rule move to `TestCurrentOrder`.
  `reportUpdateFrom` joins report windows to snapshot windows by internal id and matches them through `Window.Matches`.
  It takes desktops from the report, else the snapshot, and orders each desktop with `CurrentOrder`, the target from the report and the rest from the snapshot.
  The target desktop's zone boxes come from the apply report, so card 6's `ApplyReport` and card 1's `apply.js` now carry the target desktop's leaf tiles, with a `neve-apply` assertion.
  A new `TestFlow_Place` row checks that a reopen in key order after a 1-row apply under a panel keeps every key.
- **F5** (card 12): a paragraph lists every reader of a slug's live records and says why counting a `RolePlacement` record is right for each, with the record's full lifetime.
  The readers that select by role are named as passing over it, and no reader gets a role check.
  The registration scope is deferred, below.
- **F6** (card 8): `Runner` gains a per-call timeout, 5 seconds in production, bounding every exec-seam call.
  A timed-out call fails the run, and step 7 still unloads and releases.
  A new `TestRunner_ScriptRuns` row covers a load answered after the bound.
- **F7** (card 12): the `UnknownWorktree` row and case are dropped.
  `RefusedTaskMode` (its own task worktree, beam file `sideways`) and `RefusedMachineMode` (its own hub, machine-layer `sideways` through `configengine.Set`) replace it.
  Each asserts a settled entry and the warning's way forward in the child's log.
  The machine row asserts no marker, since the window command's own desktop-config load refuses the same key.
- **F8** (cards 11, 12): step 7 covers exits of steps 1 to 6.
  A failed `RecordOpen` warns and the later write is still attempted, and a new row with a read-only places directory made through `permkit.Chmod` asserts that `afterPlace` ran once.
  Card 12 counts a failed `kwinplace.NewRunner` among the resolution failures.
- **F9** (card 8): `PATTERN-test-speed.md` moves to `**Edit:**`, joined by `docs/benchmarks/running-tests.md`, whose two `tier3` lines gain a desktop session.
- **F10, fixed differently** (cards 8, 12, 13): `internal/kwinplace/contract_test.go` (tier3) and `internal/reedcli/placewindowchild_integration_test.go` (tier1) are named, as the review proposed.
  Card 13's test gets the new file `internal/reedcli/handterminalwindow_integration_test.go`, not the proposed `placewindow_integration_test.go`.
  That file's comment says it runs `placeTerminalWindow`, and a card cannot target a `_test.go` header by file (`edit-test-file`).
- **F11** (card 11): window-wait queries pass what remains of the wait as `within`, the locked query passes 30 seconds, and `ReadPlaces` runs after the locked query.
  That read feeds `Anchor`, `PlanInput` and `reportUpdateFrom`, and the window-wait row asserts `within`.
- **F12** (card 13): `internal/reedcli/window.go#` replaces the two member targets, and the Intent states `openTerminalWindow`'s new signature.
  The file comment changes to name the hand-off to the placement child, and both `window.go` files are spelled out by path.
- **F13** (card 13): the package tree in `docs/overview.md` gains an `internal/kwinplace/` line, and the `internal/desktophook/` line widens.
- **F14** (card 9): `RecordOpen` sets `SettledBy` to 0, and a new row checks that a reopen with the entry's own previous pid stays unsettled.
- **F15** (card 9): the heal takes `places.lock` and re-reads with `os.ReadFile` and a JSON decode.
  `state.ReadJSON` takes the same lock and would hang (`internal/state/state.go:124-126`).
  The file is removed only if it still fails to decode, then the lock is released and the update runs again.
  The undecodable row now runs two concurrent writers and asserts both entries survive.
- **F16** (card 11): a new row has the home in the window-wait snapshot but not in the locked one, and asserts that the child waits for its parent and anchors on the parent's desktop.
- **F17** (card 12): the bound now lists every file the verb writes, the per-script temp file included, and the one window command it runs.
- **F18** (card 8): the contract test adds a second load of a loaded name, `isScriptLoaded` before and after unload, and an operator-gated `Apply` on a throwaway window, each compared with its recording.
- Overview: the probes-before-code Shared Decision now states card 1's role, and card 6's Card Index line no longer claims the scripts and recordings.

## Self-check

- fix-consistency, MEDIUM: card 6's recordings walk would meet `neve-qdbus`, which none of its readers handles. Fixed as proposed: the walk covers directories with a `journal.txt` or `kscreen.json`, and card 1 names card 8 as the `neve-qdbus` reader.
- fix-consistency, LOW: card 13 named two `window.go` files ambiguously. Fixed as proposed: both are spelled by path.
- fix-claims, MEDIUM: card 12's record lifetime left out the window command's bound. Fixed as proposed: it adds the 30-second bound and 5-second group kill.
- fix-claims, LOW, merged with fix-proposal, MEDIUM: card 12's write bound left out the per-script temp file. Fixed as proposed.
- fix-proposal, LOW: `placewindow_integration_test.go`'s file comment would go stale. Fixed with my own change: card 13's test gets its own tier1 file (F10 above).
- fix-proposal, LOW: `RefusedTaskMode` could change `TaskWindow`'s mode on a shared hub. Fixed as proposed: it runs on a task worktree of its own.
- fix-affected-set, LOW: card 12's reader list missed five role-selecting readers. Fixed as proposed: they are named as passing over a placement record.

## Disputed

None.

## Deferred

- **F5, registration scope:** card 12 now has the reader analysis.
  The run-registration Shared Decision still registers every non-prime child, which goes beyond D10's "when the opening verb runs inside a run".
  Limiting registration would cut a beam's teardown off from its child's SIGTERM, which D12 relies on, and no signal tells an opening verb it runs inside a run.
  The other way forward is the parent's `lym loom decision add` stating that every non-prime child registers, for the Shared Decision to cite.

## Rulings

- **F4:** the parent's added decision "A first card runs the KWin probes" asks for a card ahead of every card that rests on a KWin fact.
  That card runs the read-only probes, takes the recordings cards 6 and 8 name with their `source.yaml` lines, records each outcome in its change description and halts on a contradicted fact.
  It also asks the operator, through the parent, for the moving probes before any later card starts, and it leaves cards 6 and 8 without an operator step.
  Done: card 1, `kwin-probes`, was inserted through `lym loom insert-card`, with the Open-risks facts mapped to the probe that observes each.
  It records the outcomes in its commit message, halts before that commit on a contradicted fact, and gates the moving recordings on the operator.
  Cards 6 and 8 lost their recording creates and operator steps, and the overview's probes-before-code decision points at card 1.
  Two consequences, both inside the ruling:
  - each journal recording is a run of the query or apply script, so the scripts' JavaScript moves into card 1 as `internal/kwinplace/query.js` and `apply.js`, and card 6's `QueryScript` and `ApplyScript` embed them and fill in a JSON input;
  - card 1 commits the recordings, since a card's work is its commit and the files must exist for card 6's tests, so cards 6 and 8 list them under `**Uses:**` rather than commit them.
  The parent was not told: the ruling is neither mistaken nor redundant.
