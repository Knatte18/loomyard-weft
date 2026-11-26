---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Clearing the copy whenever an unlisted run has no landing memory also fires after the landing notice posted and dropped its memory on a tick past Land, so a later landed reading would post a watch line beside the landing notice; the reset is limited to a run that was ahead of Land when last listed, through a new AheadOfLand flag."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: ruling
    reason: "The parent recorded the Added after Discussion entry 'Gate-hold line caps listed processes', which keeps the 20-process cap; card 6 keeps it and now cites the entry."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The gate-hold notice is wired in card 7, so card 4 names only the [run watch] lines in docs/overview.md and card 7 names the gate-hold [dobby notice], keeping each commit's doc true."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: disputed
    ground: premise false
    rebuttal: "The stated reason holds: startReady returns an error on a failed LoadConfig, tick passes it up, and passHarness.tick fails the test on any tick error, so the row cannot run through the harness's ticks even though gateHolds runs before startReady. The row keeps its direct call, with the reason spelled out."
    evidence:
      - internal/dobbyengine/starts.go:93
      - internal/dobbyengine/loop.go:720
      - internal/dobbyengine/passes_test.go:245
  - id: F13
    disposition: fixed
---

# Plan round 1 — fixer report

## Changes

- F1 (card 3): `clearDoneRuns` also leaves a run with no watch level read yet while both `RunWatch.Level` and `RunWatch.Post` are set; `TestClearDoneRuns` gains a row whose `Level` fails and then reads `stops`.
  The row says the run clears "by the tick after at the latest", since the watch posts on the same tick it first reads the level, before the clear runs.
- F2 (card 3): step 3 sets `StatusSeen` and `ChildSeen` only after a `ReadChild` that returns no error; the Memory paragraph says "last successful child read"; the failing-`ReadChild` row goes on to the next tick, where the child is read again and its stop posts.
- F3 (card 3), fixed differently: `RunWatchState` gains `AheadOfLand`, set on every listed tick, and the copy is reset only for an unlisted run that was ahead of Land and that `State.LandNotices` no longer remembers.
  Rows: the review's unlisted-then-landed case posts its landed line; a run whose landing notice posted past Land, then unlisted and listed landed, posts no watch line.
- F4 (card 3): the copy is now step 1 and runs before the level read; a run with no level read skips steps 3 to 5; new row for a level first read after Land with land_notice on.
- F5 (card 1): `TestCheckLocalValues` is dropped; `internal/configreg#TestPrepareRunSettings` is under Edit with a `run_watch: rows` admitted row and a `run_watch: all` refused row.
- F7 (card 3): `internal/dobbyengine/doc.go` moved from Extends to Edit; the done-run-clear bullet and `clearDoneRuns`'s doc comment name the new holds.
- F8 (card 1): `internal/boardcli#storeVerbs` under Edit, `internal/boardcli/doc.go` under Uses, and an Intent sentence for the `run_settings` help line.
- F9, fixed differently: card 4 edits `docs/overview.md` to name the `[run watch]` lines, and card 7 edits it to name the gate-hold `[dobby notice]`.
- F10 (card 5): a Bound sentence on a nested gate run's marker.
- F11 (card 3): `daemon` gains `watchRecordFailed`; the watch step sets it on a failed read or write of the state record, and `clearDoneRuns` clears no run on that tick.
- F13 (card 1): `LoadConfig`'s doc comment names both range refusals.
- Self-check (card 3): the ImpactSummary names the three new clear holds.
- Self-check (card 1): `internal/boardcli_test#TestHelpSchema_LeafCommands` under Edit; its board upsert row also wants `beam.run_watch`.

`lym loom validate-plan` answers `ok: true` after the changes.

## Self-check

- fix-consistency, MEDIUM, card 6 failing-LoadConfig row runs through ticks that fail the test: fixed. The row's direct call is restored with its reason, and F12 is disputed.
- fix-consistency, MEDIUM, F3's copy reset fires after a posted landing notice: fixed differently, through the `AheadOfLand` flag (merged with the fix-proposal finding).
- fix-consistency, LOW, F1's hold disagrees with the nil-`Post` seam rule: fixed. The hold needs both seams set (merged with the fix-claims finding).
- fix-consistency, LOW, ImpactSummary says "one hold": fixed.
- fix-claims, LOW, F1's hold with a nil `Post` holds a run forever: fixed. Duplicate of the fix-consistency finding.
- fix-proposal, MEDIUM, F3's copy reset sends a landed line that should be skipped: fixed differently. Duplicate of the fix-consistency finding.
- fix-affected-set, LOW, `help_test.go` pins the help's key list and card 1 leaves it out: fixed as proposed.

## Disputed

- F12, premise false: the review says the row's reason is false because `gateHolds` runs before `startReady`.
  But `startReady` returns an error on a failed `LoadConfig` (`internal/dobbyengine/starts.go:93`), `tick` returns it (`internal/dobbyengine/loop.go:720`), and `passHarness.tick` fails the test on a tick error (`internal/dobbyengine/passes_test.go:245`).
  The row keeps calling `d.gateHolds` directly, and it now states that whole reason.

## Deferred

None.

## Rulings

- F6: the parent recorded the Added after Discussion entry "Gate-hold line caps listed processes", which keeps the cap at 20 descendants and 20 marked processes.
  Card 6 keeps its cap, its `gateHoldProcessMax` Create line and its row, and its cap sentence now cites the entry.
  The ruling needed no flag to the parent.
