# Fixer report — discussion round 1

Every finding's premise was checked against the code at HEAD and holds; every finding is fixed.

## Changes

- F1 — fixed, partly differently. D5's Bolt set-aside is now by changed `board.json` entry and `design-<slug>.md`, with `README.md` ignored; the Bolt replay re-applies entry-level changes onto origin's `board.json` through a Bolt entry merge seam over `internal/boardengine` (D1), since `boardengine` imports fabricengine. Added the two-machine acceptance criterion. Not made as proposed: a same-field conflict on Bolt is no stop; origin's value wins and the local commit is kept under a set-aside tag with the lost values reported. A Bolt stop would have no way forward, since every `lyx board` write pulls Bolt first and meets the same replay (self-check, fix-way-forward 7).
- F2 — fixed as proposed: D3 step 2 defines removed from origin by the pre-fetch tracking ref or the merge base; an entry only local Bolt holds is never ended (D3, D6); criterion added.
- F3 — fixed as proposed: D4 treats a pair under a live hub pause as not live; D7 maps a live pair origin moved past or diverged from to `stopped`; D8 wake states it; criterion rewritten.
- F4 — fixed as proposed: D5 takes the ended set from its caller, with the reconcile passing D3's set and `Fabric.Pull` and the push retry reading the origin Bolt tracking ref; the Finalize-entered exclusion applies inside D5; criterion added; ledger Q21.
- F5 — fixed as proposed: D8 cleanup runs judge-only without `--apply`; criterion added.
- F6 — fixed as proposed: D4's prime bullet uses one try-acquire of the prime lock and one read of the parent lock record, no ticket, busy at once; the PATTERN-parent-lock constraint line follows it.
- F7 — fixed as proposed: D1 names fabricengine's ownership and the injected seams.
- F8 — fixed as proposed: D10 adds the `notified` flag, set only once posted, and a later failure mails while it is unset; D8 orch start prints outstanding records; criterion added.
- F9 — fixed as proposed: set-aside commits and D6's weft-tip tag both use the local `set-aside/<slug>/<sha>` prefix; Auto-mode assumption and Neve's criterion updated; ledger Q22.
- F10 — fixed as proposed: D10's gone bullet runs one ls-remote before each weft or task-branch push, with its cost; ledger Q23.
- F11 — fixed as proposed: a deferral writes no pending flag, ends the holder's loop and writes a `deferred` record with no mail; bound named; criterion rewritten.
- F12 — fixed as proposed in D6.
- F13 — fixed as proposed in D7 and D8's wake bullet; criterion extended.
- F14 — fixed, partly differently. Rewritten and rewound code branches are now their own outcomes and both stop their entry, rather than reported rewritten and left alone, or reported ahead with a warning. Left alone, an `active` entry drives on an unrecovered warp, and the detached push carries code, so a warned ahead branch still restores the dropped commits (self-check, fix-proposal 1–2, fix-way-forward 2–3).
- F15 — fixed as proposed: the destruction-chokepoint constraint covers only the replay's abort and restore.
- F16 — fixed as proposed in D8.
- F17 — fixed as proposed in D9's bound.
- F18 — fixed as proposed in D10.
- F19 — fixed as proposed in D4 and D10.
- F20 — fixed as proposed in D10; criterion extended.
- F21 — fixed as proposed in the support log's Q8.
- F22 — fixed as proposed in the Goal.

The support log also gains a Round 1 entry and ledger lines Q21 to Q23 for the picks the fixes made.

## Self-check

- fix-consistency 1, MEDIUM, dry run cannot see a D3 stop — fixed: the dry run reports a diverged Bolt or prime records branch as needing a replay and refuses nothing; criteria updated.
- fix-consistency 2, MEDIUM, D4 bound "never waits on a lock" contradicts the records locks — fixed: the bound names only the prime lock and parent locks.
- fix-consistency 3, LOW, live-pair criterion stale against D7 — fixed: the criterion states the active and stopped split.
- fix-consistency 4, LOW, wake ordering of parked pushes only in D10 — fixed: D8's wake bullet states it, worded per fix-claims 2.
- fix-consistency 5, LOW, Q4 not marked superseded — fixed: Q4 points at Q22.
- fix-claims 1 and fix-proposal 1, MEDIUM, the rewind test misses this machine's own rewind — fixed: D4 states that this machine's own rewind moves the tracking ref, and that a failed checkout rewrite is `ResetPairCode`'s error, which converges on re-run.
- fix-claims 2 and fix-way-forward 4, MEDIUM, nothing respawns pending flags at wake — fixed: D10 and D8 now say a push parked under a pause carries its pause slug and resumes only at the lift, which the daemon makes after the wake reconcile.
- fix-proposal 2 and fix-way-forward 2, MEDIUM, `rewritten` leaves the entry `active` with no caller of `Fabric.Pull` — fixed: `rewritten` stops the entry, and its way forward is `lyx fabric pull`.
- fix-way-forward 3, MEDIUM, a rewound branch's warning names no action and the next push restores the commits — fixed: `rewound` stops the entry and names `ResetPairCode`'s ways forward.
- fix-proposal 3, MEDIUM, the dry run breaks the same-candidates contract — fixed: D8 states that `--apply`'s segments compute from the reconciled state and may list fewer, and that the dry run's report names each move and teardown behind the difference.
- fix-proposal 4, MEDIUM, `ended` at Merge-In or Finalize is unhandled — fixed: a Stuck with Halt naming the landing or removal; criterion added.
- fix-proposal 5 and fix-way-forward 1, MEDIUM, `deferred` has no way forward — fixed: D10 names `lyx fabric push` in the branch's worktree.
- fix-proposal 6, MEDIUM, a paused pair can have a re-prompted writer mid-step — fixed differently: added to the `tender-pause-holds-reprompt` open risk. Restoring the live guard under the hub pause would undo F3, whose premise holds.
- fix-proposal 7, LOW, a local change to an entry origin removed is undefined — fixed: on Bolt origin's side is kept and the local commit set aside with its lost values.
- fix-proposal 8, LOW, the per-push ls-remote covers Bolt and leaves its own failure unclassed — fixed: Bolt's pushes skip it, and its transport failure is the push's transport class.
- fix-way-forward 5, MEDIUM, the ended-and-busy notice is often unreachable and leaves the pause entry behind — fixed: the notice names `lyx batten pause <slug>` and then `lyx cleanup --apply`, and D6's teardown lifts that pair pause entry.
- fix-way-forward 6, LOW, `lyx resume <slug>` alone resumes onto the stop — fixed: the wake notice names each stop's way forward first.
- fix-way-forward 7, LOW, the Bolt entry-conflict stop has no way forward — fixed: Bolt no longer stops on an entry change, as F1 records.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
