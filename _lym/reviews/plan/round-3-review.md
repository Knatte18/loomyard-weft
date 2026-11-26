---
verdict: APPROVED
prior:
  - id: F14
    status: resolved
---

# Round 3 review

F14 is resolved.
Card 3 keeps the level in the run's watch entry, reads it again past Land only while the entry holds none,
holds the done-run clear for a run with no level,
and plans two rows whose level differs between ahead of Land and past Land.
No new BLOCKING defect sits on a hunk the F14 fix changed.

## Focus notes

- The task's own level from first listing to the landed reading: once read ahead of Land, it is the one every later judgement uses.
  It survives a dobby restart in the state record.
  It survives an unlisted tick, since the entry is dropped only when batten's status is gone and no line is owed.
  A run whose level never read ahead of Land is judged by the inherited value, which nothing past teardown can improve on.
  Not a defect, bar the narrow window listed under Rejected.
- Clear, state entry, planned rows and `doc.go` text: they agree on the level kept from ahead of Land and on the hold for an entry with no level.
  The third `TestClearDoneRuns` row seeds the entry shape the hold reads.
  The two level-change rows of `TestRunWatch` each fail if the level is read again past Land.
  Not a defect.
- The decision record's D4 entry list and D2 do not mention the kept level, as the fixer report says.
  The record is the answer key, so this is no finding against the plan.

## Rejected

These are real but are not raised: an incremental round raises a new finding only at BLOCKING on a changed hunk,
and none of these sits on a hunk the F14 fix changed.

- Card 3's `WatchReading` (`_lym/plan/03-run-watch-engine.md:50`) has no field for the child's history length plus repeats, or for its modification time.
  A tick that keeps the child part while batten is running has nothing to compute `Episode` from.
  One such tick is a running Worktree-Teardown after the child status vanished.
  An implementer who puts 0 there changes the episode of a done child, and under `stops` posts the extra line the acceptance criteria forbid.
  Of the items here, this is the one to pass to the implementer: the kept child part needs its own stamp and modification time.
- Card 3 decides whether to read the level from the `PastLand` of the tick's listing, but calls `Level` later in the same tick,
  after the pass, notice and landing-notice steps.
  A batten pass that finishes Land and removes the task worktree in between makes that read answer the inherited value,
  which is stored and kept.
  The window is narrow.
- Card 4's `Post` stamps autonomy when it posts, through `orchcli.WithRunAutonomy`.
  For a landed line posted after Worktree-Teardown, `hubgeom.Autonomy` reads the parent's chain, not the task's own value.
- The `watchRecordFailed` hold in card 3 has no planned row that reaches it.
- Card 6 groups slots by PID.
  A report step of the same `lym gate test` process holds a slot with no tier and no marker (`internal/gatecli/redundancy.go:431`),
  and the card does not say which slot gives the line its tier, site and marker.
