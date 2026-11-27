---
round: 2
exclude_lenses: []
focus:
  - "D5 with D6, local landing against origin: walk every origin read of a local landing (the owner checks' run-scope reconcile in internal/landingshed/finalize.go and finalize_owner.go, syncParent's fetched parent-tips read, handedOverStop before the parent push, records push and done write) with origin unreachable. Do the Goal, D5, D6 and their acceptance criteria state one consistent outcome that PATTERN-owner-machine allows?"
  - "D5 with D1, a second landing and push ordering: with an earlier landing's parent push still owed, what does the next landing on the same parent do in syncParent's parent-ahead case, and what mechanism keeps the board done write (markTaskDone, whose board write spawns the board sync) and Finalize's records commit (pushed by the pair's pusher) from reaching origin before the parent?"
  - "D5, D3 and D13, owed state after a crash or refusal: is 'the landing target owes a parent push' defined once and read the same way by the past-the-merge read, the pull-request check and the landed-pair teardown, including a Finalize killed between the local merge and the owed record, a refused landing push, and git or pushes turned off?"
  - "D3, D4 and D7, the ledger signals: with a refused-push record standing, and with a run waiting on a network outage while nothing is owed, do the bar's minutes, the summary and the all-clear report the true state, and what clears each?"
  - "D6, the network wait: check which lock the wait record is judged against (shedrun.RunLock is taken by the step; shedrun.DriverLock is the driver's), whether the step process or the driver writes the halt note for a waited stop, and whether permanent faults classified as network-class are bounded."
  - "D1 and D2, push paths and cadence: is the rule for which pushes move to a per-target pusher complete by kind, including lym fabric commit's detached push; can one branch (a beam parent's code branch) get two targets; and do the pusher's existing retry loops stay within one push per interval in force? Does the declared PATTERN-pair-remote text agree with every push the record keeps synchronous?"
  - "D7, the reset guard: is the git behaviour the reachability rule rests on (branch reflog contents and expiry) pinned, does every refusal and stop name a way forward, and is each caller that moves an origin branch settled?"
  - "D9, removal of leftover ignored files: how are the base tree's ignore rules evaluated and where is that pinned, which commit is the base on a resume or a fast-forward merge-in, does removal run before the pre-merge-in clean check, and does the bound state what it deletes?"
  - "Acceptance criteria: does each observed fault in the Goal, the failed-push mail of issue #238 included, have a test stated to fail on the merge base, a reason no test seam exists, or a statement that no bug is fixed?"
---

All lenses of the latest round reported something at MEDIUM or worse, and their areas are in the focus above, so none is excluded.
