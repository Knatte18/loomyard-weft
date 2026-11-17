```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals the fixer carried out: F1 and F3 as proposed, and F2 as ruled by the parent.

**MEDIUM: F3's shared predicate pulls a leftover directory into every catch-up and can stall the dobby**
- **Changed text:** card 20 line 63, `DobbyPairSlugs` "whose pair stands here (the pair's worktree path exists, the predicate `RunView.PairPresent` reads)". Card 21 says the same: "`DobbyPairSlugs` reads ... by the same predicate as `PairPresent`".
- **Case:** a pair directory is left standing but is no longer a registered worktree, for example because its `.git` link points to a pruned gitdir.
  1. Under the new predicate this directory is in the catch-up's scope.
  2. Card 20's `FetchRun` therefore runs `fabricengine.FetchPairTips` of that pair instead of fetching the prime's records branch, as it does "for a run with no pair here".
  3. `FetchPairTips` (`internal/fabricengine/codetipsettle.go:20-28`) runs `git fetch` in `RecordsWorktree(pair)` and in `pair.WorktreePath()`. In a directory that is not a git worktree, that fetch fails.
  4. The failure is not a transport failure, so the catch-up records `failed`.
- **Effect:** `handedOver` bounds only the `hand_over` trigger. A fresh start, a seen sleep, or a `PassExitOriginWait`/`PassExitOwnedElsewhere` exit still owes the catch-up. A failed catch-up stays owed, and card 21 says "While the catch-up is owed the tick spawns no pass". So one leftover directory stops every run of the dobby until the operator removes it.
- **Origin:** before the fix the predicate was unstated, so this failure comes from the proposal.
- **Fix direction:** decide scope by the hand-over's own test, a registered linked worktree as `Teardown.TaskWorktreeGone` reads it, and keep `PairPresent` only for the trigger. Alternatively, state that `FetchRun` handles an unregistered directory as "no pair here".

**LOW: F2's ruling leaves an `isBeam` read failure unspecified**
- **Changed text:** card 23's bring-up paragraph: "`endDobby` is set when the entry is a beam, read through an `isBeam func(slug string) (bool, error)` seam".
- **Case:** the bring-up reads the board owners and the read fails. The card says what happens for a nil `endOwned` (skipped and warned) and for a failed end (the "could not be ended" warning). It says nothing for an `isBeam` error, so the implementer must choose between:
  - warning and skipping the entry;
  - ending it with `endDobby` false, which leaves a beam's dobby running here;
  - failing the start.
- **Tests:** `TestStart_ReconcilesTheHubFirst` has no row for this case either.

**No findings on F1.** Moving the `bolt-hold`-only refusal ahead of `req.Clear` is consistent with `Resume`'s existing order (verdict, then clear, then lift) and with card 22's pause intents. I found no case where refusing before the clear is wrong.

Tool calls:
1. A grep of cards 18 and 20 for how `FetchRun` treats a run with no pair here, plus `FetchPairTips`' body. My context did not have either, and I needed them to build the F3 case.
