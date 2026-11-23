# Webster stops reach the parent; reset records the new code tip

## What changes

Fixes three defects behind issue #207, where an autonomous tapestry run stalled silently at Webster after an amended plan.

- **A judged blocked stop now reaches the parent while a batten watches.**
  At a hand-back, the shed driver now sends its `Run <id> stopped; stop report …` line for a blocked stop it asked the tackler or the arbiter to judge, whatever that agent answered, even while a batten watches the run (`internal/sheddriver`).
  Before, a watched hand-back sent only a relayed notice, a question or a friction-failed done, and left every other stop to batten's Run-Shed notice.
  Batten holds that notice while the arbiter may still settle the stop, and every Webster block goes to the arbiter first.
  A block the arbiter handed on with no question, or refused, therefore reached no parent.
  Awaiting stops and stops no agent judged keep their old notices.
- **The Plan gate refuses an overview edit outside its Card Index once a batch has begun.**
  `ValidatePlan` (`internal/loomshed`) adds a blocking `overview-frame-edited` finding when the run record holds a begun batch and a recorded overview frame, and `00-overview.md` no longer has that frame.
  The Plan rows that apply an `--amends-plan` decision are now sent back to keep to the Card Index, instead of passing the gate and then blocking at Webster's auto-rebaseline, which refuses the same edit.
  A record with no recorded frame, or with no begun batch, reports nothing.
- **`lym webster reset` marks the run's transition at the new tip.**
  In hub mode the reset's records commit now carries `Run-Transition: <worktree run>` beside its `Warp-SHA` (`internal/webstercli`).
  The run's last transition therefore names the commit the reset moved to.
  Before, it still named the dropped tip, so `lym loom resume` refused with "the code tip … is not on origin yet", and past `runfsm.CodeTipGrace` the dobby would have returned the run to that transition.
  To support this, `fabricengine.Fabric` gains `CommitWithTrailers`, which commits as `Commit` does and ends the weft message with the caller's trailers after `Warp-SHA`.

## Design decisions

- The driver sends its notice whenever an agent was asked to judge the blocked stop, not only when the agent recorded an attempt.
  An arbiter that refuses the stop, for example with its spawns spent, records no attempt, and that path was silent too.
  The run's parent confirmed this route, since the incident's own records were not on this machine to show which path stayed silent.
  While batten's hold ends without the arbiter's hand-on, the parent may get both the driver's line and batten's notice for the same stop.
- Defect 2 is fixed in the Plan gate rather than in the plan stencils, so the gate and Webster's rebaseline check the same frame, as the parent agreed.
- Only `webster reset` writes the transition.
  `lym loom goto` writes only the status file and never moves the code tip,
  so once the reset records the new tip, goto has no stale tip to correct and is left unchanged.

## Tests

- `TestDriveNotices` gains rows for a watched blocked stop an agent judged (run-id line sent), one no agent judged (nothing), and a watched awaiting stop an agent judged (nothing).
- `TestGoDriver_Settle` gains a `watched` field and two rows: a watched blocked stop the arbiter judges first and hands on with no question, and one the arbiter refuses; each sends the parent one notice.
- `TestNewPlanGate` gains four rows: an overview edited outside its Card Index after a begun batch fails, and an unedited overview, an edit before any batch began, and a record with no frame all pass.
- `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` runs a real `reset --to batch-start` and checks that the records HEAD message holds `Warp-SHA: <start>` and `Run-Transition: <worktree>`.

## Merge danger

**Door:** two-way
**Blast radius:** loom
