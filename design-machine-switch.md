# One verb to take work over on another machine, leaving every run in a known state

`machine-switch` · Task · feature, fabric, dobby, batten · running · Discussion-Write

## Seen (operator, 2026-10-31)

Moving work from one machine to another is a string of hand steps, and afterwards the operator cannot tell what state the runs are in.
On 2026-10-31 the operator moved from OSL-1046 to Neve: `lym fabric pull`, `lym s`, a `lym adopt` whose tier0 check mailed the orch, and more by hand.
Task `dobby-land-notice-needs-flag` had started on OSL-1046 on 2026-10-30 and stood at `running · Code-Bouncer`, unlanded.
After the move, dobby on Neve reported its driver strand dead, then a fresh Go driver took it on, and neither the operator nor the orch could say at once whether the run was finished, should run or should not.

The run went on because `lym adopt` resumes it: step 6 of its help resumes each moved run not done as a detached first pass, so adopting is also starting.
Bolt itself is reconciled with origin first by each of `lym fabric pull`, `lym adopt` (its step 1) and `lym beam start`, so the board was current; what was missing is a view of the runs.

## Design (operator, 2026-11-01)

One verb takes a beam, or root, over onto this machine and leaves it in a known state, in place of the hand steps.
- The unit is the beam: the operator names the beams (or root) to take over, and lym supports one beam owned and run on one machine while another beam is owned and run on another.
- The verb does every step the move needs, each idempotent: sync Bolt, sync warp and weft of the named beams, adopt them, settle what conflicts on the way, and start each beam's dobby and orch as a pair.
  Which steps those are is for this task to find out from the code (`lym fabric pull`, `lym adopt`, `lym beam start` and what they leave to the operator today); the operator does not list them.
- The old machine releases nothing: it loses ownership when the new one adopts, and the operator never works one beam on two machines at once.
- It starts no run by itself: every adopted run stays held until the operator lets it go on, so nothing runs that the operator does not expect.
  The hold is released per beam, by a verb; a button on reed's action row in that beam's session calls the same verb once note `reed-status-buttons` is built.
- It prints one line per run: landed, running here, held for a go-ahead, or owned by another machine, with the row it stands at.
