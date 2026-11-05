# Fixer report — discussion round 1

F7 is `decision` class and is left to the parent.
Every other finding is fixed in `_lym/discussion/decision-record.md`, and round 1 is logged in `_lym/discussion/support-log.md`.

## Changes

- F1 (BLOCKING), fixed as proposed: D11's third paragraph now reads the board status before Beam-Await's own write. If the hold names Finalize or a retired row and no land request stands, the entry reads the ready record. When the record stands, the entry runs the accept's re-check itself. When it does not, the entry lifts the hold and starts the dobby whatever the marker says. D10's Bound gives the same rule for a goto, and the claim that the dobby lands the beam again is gone. A Constraints line names `dobbyengine` `State.Land`. The D10 and D11 criteria are updated.
- F2 (MEDIUM), fixed as proposed: D2 adds the fork precondition in `internal/battencli/beamfork.go`, which refuses a member whose beam's ready record names that beam and names `lym batten cancel <member> --remove`. D2's Bound names both guards, D3's and D10's Bounds cite D2's guards, and D2 gains a criterion for a hand `lym batten run`. Scope is updated to match.
- F3 (MEDIUM), fixed as proposed: Beam-Await stamps each completed wake in a never-tracked wake record. The repair count restarts at zero only when that stamp is at least ten minutes after the `At` of the key's last restep settle record. A stated Bound and a criterion cover a loop that crashes after each first wake.
- F4 (MEDIUM), fixed differently: D3's refusal 6 now refuses a merged pull request whose merge lacks beam commits and names `lym beam pr close`, then landing without one or `lym beam pr open`. The proposed change I did not make is the D8 halt "as PR-Review does today". Once the ready record stands, every beam verb refuses (D5, D6), so a halt at Finalize would have no step that works. D8 instead lands such a pull request as with none recorded. Finalize pulls the parent first (`syncParent` in `internal/landingshed/finalize.go`, after the pull-request decision), so the merge-in carries GitHub's merge and the landing carries only the commits the merge lacked. The D3 and D8 criteria are updated.
- F5 (MEDIUM), fixed as proposed: D8 names `prMergeRetryWindow` and RefusalPending. A refusal still pending past the window, and every RefusalOther, halt naming GitHub's message, `lym beam pr status` and `lym loom resume`, never `lym batten cancel`. A Constraints line cites `classifyRefusal`. The D8 criterion is updated.
- F6 (MEDIUM), fixed with the review's alternative: D8 gains a bullet. With nothing in D4 and an open pull request in the run's scratch record, whether the old PR-Review or a halted Finalize put it there, Finalize merges that pull request without editing its text. D13 says D8 lands it. The D8 criterion is updated. The first proposal, a carry into D4 at Finalize, was tried and reverted; see Self-check.
- F8 (MEDIUM), fixed as proposed: D13 commits the carry and the ready record's removal together through `fabricengine.CommitAnchoredPaths` over the landing directory. The D13 criterion is updated.
- F9 (LOW), fixed as proposed: `close` writes the land-without comment only where the resolver answers `on` or `off`. On a `merge` parent, `close` reports that Finalize still lands through a pull request. On a required parent, it reports that `lym beam ready` refuses until `lym beam pr open`. The D6 criterion is updated.
- F10 (LOW), fixed as proposed: D3 names D13's move among the steps that end the ready record.
- F11 (LOW), fixed as proposed: D13's move and its criterion also remove the reopen request.
- F12 (LOW), fixed as proposed, taking the "may not" branch: D5 says `lym beam describe` owns `summary.md` and that the orch's text comes only through the entry's title and brief. D14's role text says the same.
- F13 (LOW), fixed as proposed: D11 cites the re-invoke entries in `loomshed.InterruptPolicies` under the arbiter's existing rule. The D11 criterion is updated.
- F14 (LOW), fixed as proposed: D13 and its Constraints line say the move leaves the run paused, which the driver's rule table hands back to the parent. The criterion is updated.
- F15 (LOW), fixed as proposed: D8 names the one-minute window, and D6 retries `unknown` for up to one minute. The D6 criterion is updated.
- F16 (LOW), fixed as proposed: D4's Rationale keeps only the restart and adopt reason. D8's required bullet states that Finalize keeps a pull request it finds or opens in its scratch record.
- F17 (NIT), fixed as proposed: the two restating assumptions and D2's restating Rationale clause are cut.

## Self-check

- fix-consistency, MEDIUM: D8 dropped the pull request before the parent-side landing, against D4. Fixed: it is now dropped after the landing, as always.
- fix-consistency, LOW: D13 cited `lym beam reopen`, which D10 removes, as its commit precedent. Fixed: D13 names `fabricengine.CommitAnchoredPaths`.
- fix-consistency, LOW, merged with fix-proposal MEDIUM 1 and the note fix-way-forward raised outside its lens: the carry at Finalize had no trigger, and it would also take a pull request a halted Finalize opened. Fixed with my own change: the carry is dropped, and D8 reads the scratch record (see F6).
- fix-way-forward, MEDIUM: refusal 6 refused a merged pull request that lacked no beam commit. Fixed: it passes when the merge lacks none, and the D3 criterion is updated.
- fix-proposal, MEDIUM: the D11 stamp rule wired Beam-Await into the shared driver. Fixed: the driver reads the stamp through a seam the pawl wiring tells.
- fix-proposal, MEDIUM: on a required parent, `close` pointed at a Finalize that `lym beam ready` never reaches. Fixed: it reports that ready refuses until `lym beam pr open`.
- fix-proposal, LOW: RefusalMethod was not covered. Fixed: it halts as RefusalOther does.
- fix-proposal, LOW: the fork guard's behaviour on an unreadable ready record was open. Fixed: such a record also refuses.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
