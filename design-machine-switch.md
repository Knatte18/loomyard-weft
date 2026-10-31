# One verb to take work over on another machine, leaving every run in a known state

`machine-switch` · Note · feature, fabric, dobby, batten

## Seen (operator, 2026-10-31)

Moving work from one machine to another is a string of hand steps, and afterwards the operator cannot tell what state the runs are in.
On 2026-10-31 the operator moved from OSL-1046 to Neve: `lym fabric pull`, `lym s`, a `lym adopt` whose tier0 check mailed the orch, and more by hand.
Task `dobby-land-notice-needs-flag` had started on OSL-1046 on 2026-10-30 and stood at `running · Code-Bouncer`, unlanded.
After the move, dobby on Neve reported its driver strand dead, then a fresh Go driver took it on, and neither the operator nor the orch could say at once whether the run was finished, should run or should not.

The run went on because `lym adopt` resumes it: step 6 of its help resumes each moved run not done as a detached first pass, so adopting is also starting.
Bolt itself is reconciled with origin first by each of `lym fabric pull`, `lym adopt` (its step 1) and `lym beam start`, so the board was current; what was missing is a view of the runs.

## Idea

One verb takes a machine over and leaves it in a known state, in place of the hand steps.
- It pulls warp and weft, adopts the runs and beams the operator names, and starts the dobby and orch, each step idempotent.
- It starts no run by itself: every adopted run stays held until the operator lets it go on, so nothing runs that the operator does not expect.
- It prints one line per run: landed, running here, held for a go-ahead, or owned by another machine, with the row it stands at.

## Open

- Which steps the move took on 2026-10-31 in full; the operator's account is the source.
- Whether the old machine releases its runs first, by a verb of its own, before the new one adopts them.
