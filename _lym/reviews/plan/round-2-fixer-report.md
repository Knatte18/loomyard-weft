---
dispositions:
  - id: F14
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- **F14 (fixed as proposed)**, in `_lym/plan/03-run-watch-engine.md`:
  - Memory: `RunWatchState` gains `Level string` (`json:"level,omitempty"`), the last `run_watch` level read for the run.
    The in-memory `watchLevels` map is gone from the `daemon` fields.
  - Step 2: `Level(slug)` is called only while the run is not `PastLand`, or while its entry holds no `Level`.
    It now states why: Worktree-Teardown removes the task worktree before batten reads done, so a read past Land answers an inherited value and succeeds.
    A successful read is stored in `Level` and a failed one keeps it.
    A run whose entry holds no `Level` skips steps 3 to 5.
  - Entry upkeep: the clear hold is now a run with no `Watches` entry or whose entry holds no `Level`, while `RunWatch.Level` and `RunWatch.Post` are both set.
    The entry may be absent because a run whose level never read has no entry until step 1 or card 6 creates one.
  - `TestClearDoneRuns`: the failing-`Level` row seeds an entry with no `Level`.
    The posting row's seeded entry holds `Level` `stops`, since that row's run is done and step 2 no longer reads its level.
  - `TestRunWatch`: two new rows.
    In the first, the level reads `off` ahead of Land, answers `stops` past Land and across a `restartDobby`, then the run lands, and no line posts.
    In the second, the level reads `stops` ahead of Land and answers `off` past Land and at landed, and the landed line posts once.
  - The `doc.go` sentence names the level kept from ahead of Land and its reason.
    Its done-run-clear wording names a run whose watch entry holds no level.

## Self-check

- fix-consistency, LOW: D4's list of a watch entry's contents (`_lym/discussion/decision-record.md:103-104`) omits the stored level, and D2 does not say that the level is kept from ahead of Land. Deferred: the decision record lies outside this round's write surface, `_lym/plan`.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- Self-check fix-consistency LOW: the decision record's D4 entry-contents list and D2's level rule do not mention the level kept from ahead of Land.
  The change belongs in `_lym/discussion/decision-record.md`, outside this round's write surface, so the parent decides whether to record it, for example through `lym loom decision add`.

## Rulings

None.
