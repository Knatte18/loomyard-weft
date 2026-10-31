# Fixer report — discussion round 1

Every finding's premise held against the code, and every finding is fixed in `_lym/discussion/decision-record.md`.

## Changes

- F1, fixed as proposed: the last Constraints fact now states that the step watchdog has no row exemption, with the readings `agentActivityFor` counts, and that Beam-Await and PR-Review steps with no agent activity are killed today; the second Open risks bullet is deleted.
- F2, fixed as proposed: D6's stamp is the count of the run's entries into the row, each a Done of the row whose `on_done` names it or a `goto` onto the row from another row, and a halt and a resume write none; the D6 acceptance bullet covers a flag written while halted, a `goto` away and a return with no Done between.
- F3, fixed as proposed: D6's done-flag wake stops the session first, then checks; a dirty checkout clears the flag, resumes the session and then mails it; the acceptance bullet says the session is left live and mailed.
- F4, fixed as proposed: D6 gains an entry bullet that reads the flag before the session is ensured, and a flag with a clean checkout and an intact record ends the row Done with no session; the Bound and the acceptance bullet cover a recorded session that can no longer be resumed.
- F5, fixed as proposed: D4's Bound drops the Describe leg and names the append-only, labelled, dated, committed entry from the caller rule's package doc.
- F6, fixed as proposed: the acceptance bullet says Code-Bouncer is recorded Skipped, the run goes on to Describe and no Code-Review row spawns anything.
- F7, fixed as proposed: D6's stamp bullet and acceptance bullet no longer name a reroute.
- F8, fixed as proposed: D3 no longer names `CodeReviewOff`.
- F9, fixed as proposed, first alternative: D5 gives the session the publish guard's blocked forms with steer text of its own.
- F10, fixed as proposed: D7's Bound no longer cites Run-Shed.

## Self-check

- fix-consistency, LOW, support-log Q16 still gave the Done-count stamp: fixed as proposed, Q16 now gives the count of entries into the row.
- fix-consistency, LOW, support-log Turn 6 still gave the held-turn-end mechanism: fixed as proposed, Turn 6 now names the missing wait marker and the missing row list.
- fix-consistency, LOW, support-log lens-findings line still named `CodeReviewOff`: fixed as proposed, the line names only Finalize's reroute.
- fix-way-forward MEDIUM and fix-proposal MEDIUM (merged), a damaged record with an unresumable session loops on `lym loom resume` because the entry check ensured the session first: fixed as proposed, the entry bullet now judges the record before the session and halts with the restore steps; the dirty-checkout branch alone waits for the session, to mail it; the Bound now reads "a clean checkout and an intact record", and the acceptance bullet covers the damaged record on entry.
- fix-way-forward, LOW, D5's new steer text names no step for the agent: fixed as proposed, the steer names a commit and `lym loom sampler-done` once the operator says the work is done.
- fix-proposal, LOW, a relaunch or `goto` onto the row while halted at it raises the stamp and drops the flag: fixed as proposed with its first option, a `goto` counts only from another row. The relaunch half of the premise is false, since `Relaunch` writes no entry when the target is already in effect (`internal/shedengine/goto.go`, the `Relaunch` doc), but a `lym shed goto` onto the current row does write one (`goto.go:135`), so the fix stands.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
