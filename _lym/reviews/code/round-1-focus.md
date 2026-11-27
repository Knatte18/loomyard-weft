---
round: 1
focus:
  - "internal/landingshed/finalize.go, the tip compare in landUnderLock against the tip parentMergeTip picks: for each parent-to-upstream relation (equal, behind, ahead, diverged, no upstream) and with the push skipped, settle whether the locked head can match gatedTip, what the parent sync does first, and whether any state counts lost races or loops where it should halt or land"
  - "What guarantees the landed tree passed the gate on the parent head it lands on: trace every path from a match on gatedTip to the parent-side merge, including the retry after the parent moves during landing, a freeLock rerun in the locked pass, a skipped verify of an already verified tree, and a resume from a halt"
  - "State carried from the unlocked merge into the locked pass, whose merge is then up to date: parentHeadMerged and the landed diff base, the merge record with its merged tip and overlap, the fixer attempt record, fix review state and notices; settle whether the second merge overwrites or loses any of them"
  - "Steps that ran under the lock before the first merge and now follow the unlocked merge and gate (owner checks, the pull request decision, parent pair checks, the parent sync, ConfigChanges): settle whether any case that halted or took another path before touching the task worktree now merges, runs a fixer or reports differently, and whether ConfigChanges still diffs against the parent the task forked from"
  - "gateUnlocked with no lock: the parentHoldsWork read, clearing a parked merge-in, a conflict session, the merge fixer and fix review; settle whether any of them or their callees assumes the parent lock is held or that the parent stays still, and how a pause, a lock wait or the idle release behaves there"
  - "lockHeld, lockFreed, parentMoved, gatedTip and lostRaces across passes of Call: settle that each is reset where needed, that a bound of 0 behaves as before this change, and what a hub config without finalize_lost_races reads as"
  - "internal/landingshed/finalize_unlocked_test.go against the new behavior: settle which of the upstream-tip path, a conflict or failing gate with its fixer in the unlocked pass, and an error from the new reads have no coverage here or in the existing tests"
  - "Prose that states when Finalize holds the parent lock (contracts, docs, specs, stencils, doc and inline comments, lock-wait messages, including files the diff does not touch) and _lym/landing/summary.md claims and Merge danger call: settle whether each matches the code after the change"
---

The decision record holds only the board entry, with no added entries, so the entry's Idea section is the measuring stick.
The change moves the merge and gate ahead of steps that used to precede them under the lock,
so the ordering and the state shared between the two passes carry most of the risk.
