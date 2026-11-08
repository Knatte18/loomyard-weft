# Round 3 fixer-report — discussion

Every finding's premise held against the code; all six are fixed.
Changed files: `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  D4 gains a Decision that corrects `freshRaddleEntry` to match a goto entry as `shedengine.Goto` and `shedengine.Relaunch` write it: the target as producer, outcome `goto`, no `RoutedTo`.
  The goto branch tests outcome `goto` and producer `Raddle-Write`; the Raddle-File Stuck branch keeps its `RoutedTo` test.
  The goto rows of `TestFreshRaddleEntry` are to be rebuilt the same way.
  New D4 criterion: after `lym loom goto --to Raddle-Write` on an arras run, the last proposal is moved aside and the seat starts a new session.
  D2's bullet drops "as `freshRaddleEntry` judges it".
  Scope In names the correction, and Scope Out says it is the one change a `raddle` run sees.
- **F2 (MEDIUM), fixed differently in one part.**
  D2's re-arm bullet now covers only a perch whose last verdict settled.
  A new D2 bullet accepts the goto from a perch that has not settled, with its bound: the recorded round settles again and raises its escalation again;
  below the hard cap `circling continue` reviews the new proposal, and at or past it only `circling accept` is possible, which files a proposal no round reviewed.
  The `run.go` budget point and the no-generation-start point are stated as proposed.
  Not made: the proposed sentence "a budget-halted perch therefore halts on its next Stuck, and `lym loom resume` grants one more budget".
  A perch past its budget never takes Shed's budget halt: `settleUndecided` sends an open soft-cap window to `softCapRule` (`internal/shedadapters/bouncer.go:1361-1362`),
  which accepts, escalates, or returns a `BudgetExempt` Stuck (`bouncer.go:1440`), and `episodeStuckCount` skips a `BudgetExempt` Stuck (`internal/shedengine/run.go:672`).
  The criterion at the old line 249 now names a goto from Raddle-File, and a new D2 criterion covers the escalation below the hard cap.
  D4's Bound names this as a third route that can file an unreviewed proposal.
  The support log's question ledger records the auto-pick.
- **F3 (MEDIUM), fixed as proposed.**
  D4's soft-cap sentence now counts open BLOCKING or MEDIUM ledger entries whose fix tag is not `mechanical`, a missing tag included, as `designWrongKeys` does.
  The bound adds that open LOW and NIT entries and BLOCKING or MEDIUM entries tagged `mechanical` are filed, with the judge's tag as the only guard.
  The D2 criterion and the twin D4 criterion are reworded the same way; the D4 one now includes an open LOW entry.
- **F4 (MEDIUM), fixed as proposed.**
  D6's Bound names the rows as any run status with producer half `Raddle-File`, whatever its state half, and the writers as any board writer.
  It replaces the approved-shape claim with what the admitted write can leave, so the release stays the gate.
  A new paragraph records `Call`'s two own-filing tests (store done, or staged hash plus `AlreadyFiled` over the current proposal's slugs),
  and decides that Raddle-File, finding the entry already a project with neither test holding, files nothing and halts for the parent with `lym batten cancel <slug>` as its way forward.
  New D6 criterion for that halt; the support log's question ledger records the auto-pick.
- **F5 (LOW), fixed as proposed.**
  D6 names the lock's input as the `after` list `checkRunLocks` receives, the filed parts as its added entries in their sequence in `after`,
  defines the `order` check against that sequence with each part after its in-family dependencies, and states that the lock recomputes no canonical order.
- **F6 (LOW), fixed as proposed.**
  D6's retry rule writes the mark and clears the status at the start of `Call` whenever the converted entry is found filed as its own, store done or not, then writes the store when it is not done.
  It adds that `finish` skips `EndEntry` on every path in the family outcome, and a criterion for a store-done filing with no mark.

## Self-check

- fix-consistency, MEDIUM, D2's first bullet still claimed every fresh entry re-arms the perch: fixed as proposed, the bullet now covers only a settled perch.
- fix-consistency, LOW, Scope Out did not mention the `raddle` goto change: fixed as proposed.
- fix-consistency, LOW, D4's Bound named two routes and missed D2's unreviewed-accept route: fixed as proposed, D4 names a third route.
- fix-proposal and fix-way-forward, MEDIUM (merged), `circling continue` is unavailable at the hard cap: fixed as proposed, D2's bullet splits below and at the hard cap, and the criterion is restricted to below it.
- fix-proposal, LOW, the D6 halt called the run's own pre-goto filing one the run did not make: fixed as proposed, the halt names a filing not made from the current proposal, and D6 lists that case.
- fix-claims, MEDIUM, the budget-halt claim does not hold: fixed as proposed, the claim is removed (see F2).
- fix-claims, LOW, only one goto row carries `RoutedTo` `Raddle-Write`: fixed as proposed, D4 now says the rows carry a `RoutedTo`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
