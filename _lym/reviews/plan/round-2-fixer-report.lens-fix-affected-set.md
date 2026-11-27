```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran each finding's search where the premise covers a set and checked that the fix covers every member.

Unchecked list: F2 (I only spot-checked it: the pushes of a beam's code branch that skip the owner reads, see the LOW below), F5, F8, F9 and F10 have single-site premises and needed no set search.

**BLOCKING.** The F3 fix covers only some of the readers of an unrecorded `Lym-Landing` commit. It concerns card 10's lost-ledger hold paragraph ("a task pair push that finds no hold … reads the run's parent through `LandingCommits` … sends no records commit past origin, until the parent target's next pass re-records the landing") and card 10's new beam ceiling ("or a commit `LandingCommits` lists, and `hubLandDeps`' `PushBeam` refuses on either").
- F3's fix adds the board-`done` skip to `LandingStands`, to the landing target's `Ahead` and to `settleParent`'s entry. Two other readers of the same trailer did not get it: card 10's `recordsPushCeiling` trailer read, and card 10's beam code ceiling and `PushBeam` refusal.
- Take a landing made with pushes off and git on. Card 12 writes no entry and makes the `done` write itself, and the commit stays past origin.
- Once pushes are on again, every push of that pair, its removal's records push included, stops short of the pair's records commit for good. Card 11's re-record skips a run whose board entry is `done`, so the "next pass re-records" never happens.
- For a beam parent, the beam's ordinary code push stays capped below that commit and `lym beam pr` refuses for good. So the pushes-off case still leaves work that is never pushed, against D11.
- Evidence: the grep shows the trailer read at `10-parent-push-git.md` lines 51–52 and 60, with no `done` skip. Compare `11-parent-pusher.md` step 1 ("whose board entry is not `done`") and `19-teardown-hands-to-dobby.md` line 64.

**LOW.** The F6 fix does not cover two writers of a pair entry. It concerns card 12's `landLocally` ("sets the push-now marks of the parent's and the pair's targets", line 112) and card 11's `WakePair` (step 1).
- Both write a pair target's entry, and neither card says the write sets `Worktree` and `Slug`.
- Card 6's rule covers the mark only when it goes through `pusherCmd --push-now`, and neither card says which path it takes.
- Evidence: the grep of entry writers finds `12-finalize-local-landing.md:112` and `11-parent-pusher.md:63`.

**LOW, outside the changed hunks, for information.** `HandOverTips` pushes a pair's owed code tip with `PushCommit` or a lease move (`internal/fabricengine/handover.go:290`). For a beam pair, that push could carry landing commits with no owner read. No card caps it. The F2 fix covers `PushRun` and `PushBeam` only.

The F7 set is fully covered. The grep for `push-now`/`PushNow` finds only the landing (card 12), settle and `WakePair` (card 11), and teardown (card 19) sites, plus card 23's doc line.

The F1 set is fully covered. The only other `PushCommit` in `fabricengine`/`pairteardown` is `handover.go:290`, and that pushes the code branch, not the records branch.

Calls:
1. grep of the plan for `push-now` and `LandingCommits`/`Lym-Landing`, to find every member for F7 and F3 beyond the cards already in context.
2. grep of the plan for ledger entry writers, and of the code for `PushCommit`/`.Push(`, to find the members for F6 and F1.
3. `sed` of `internal/fabricengine/handover.go:255-300`, to see which branch the second `PushCommit` pushes.
