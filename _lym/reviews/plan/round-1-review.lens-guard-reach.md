```yaml
lens: guard-reach
outcome: reported
```

**guard-reach lens.** For each guard, refusal, branch or test row the cards add, I checked whether a real entry point can reach it and whether the guarded site recurs unguarded.

## Per guard: reachable or not

**Card 1**
- `SetOutcome` on an absent slug errors: reachable only by hand, and the row pins it.
- `Backstop` seam error keeps the mark: reachable. Under an absent origin ref it never clears (finding 3).
- `Backstop` member with no entry: reachable only by hand.
- `Backstop` mark with no held commit: reachable when a leader dies before the move.

**Card 2**
- `ErrInvalidSHA` on a non-SHA tree: reachable, row pinned.

**Card 3**
- `BuildLandingChain` unresolvable ref: reachable.
- `BuildLandingChain` `Held` and `Conflicted`: reachable for members. For the leader's own link, see finding 1.
- `MoveCodeTo` head not at `base`: reachable, by an operator or the parent pusher's reset.
- `MoveCodeTo` tracked dirt: reachable.
- `MoveCodeTo` detached head: reachable by hand.
- `MoveCodeTo` tip not descending from `base`: not reachable from `landPrefix`, since the chain is always built on `base`. The test row pins it as a fail-loud guard.
- `NewestLandingOf` 500-commit limit: reachable in its test.

**Card 4**
- `pushClassified` code-side hold-off: reachable. `PushPairAnchored`'s code side passes `headOrEmpty`, and `LandingCommits` lists none while detached (`landing.go:132-140`), before reaching `pushClassified`.
- `readLocalBranch` marker branch: reachable through `RunCommitsAhead`, `TipsOnOrigin` and `unpushedUnder`.
- `RefusedPushStands` marker branch: reachable through `TargetRefused`.
- `parentHoldsTaskWorkAt` marker branch: reachable through `prepush.go:78`, `pairteardown/runread.go` and `runpusher`'s `WorkUnderOtherCommit`.
- Deleted-branch marker error: reachable only by hand.
- `RestoreDetached` stale marker: reachable when a leader dies between its restore and the marker's removal.
- `RestoreDetached` dirty or no-marker return: reachable.

**Card 5**
- `mergeUnlocked`'s held early return: reachable.
- Removal of the gated-tip recheck: bounded, since the gate now runs only under the lock.

**Card 7**
- `joinQueue` skip on a GitHub need: reachable.
- `joinQueue` skip on a failed `Mode` read: reachable, but the "lands alone" in its row is not (finding 2).
- Queue directory that is a file: reachable.
- Backstop error: reachable. Its row's "runs no merge" contradicts the card's own `Call` order (finding 4).
- `landed` held while `parentHoldsWork` reads false: reachable.
- `landed` with a commit the parent no longer holds: reachable.
- `failed` and `conflict` outcomes: reachable.
- `mergeNeedsSession` precheck: reachable for merge-ins under the lock, but it does not cover the leader's chain link (finding 1).
- `Queue.Remove` failure keeps Done: reachable.

**Card 8**
- No-entry leader: reachable.
- `MemberDeps` failure: reachable.
- PR-routed leader: reachable.
- Batch of one: reachable.
- Restore failure into Stuck: reachable on untracked dirt the gate left.
- Member checks: reachable. `JudgeOnly` exists (`originreconcile.go:558`, `originreconcileended.go:120`).
- Rebuild bound and land-alone fallback: reachable.
- `MoveCodeTo` false, and its `MergeGuardError`: reachable.
- Leader's own link Conflicted or Held: reachable but undefined (finding 1).

**Card 9**
- Unjudged result halts: reachable.
- Culprit k ≥ 1: reachable.
- Leader as culprit at k = 0: reachable, but only if chain commit 1 is the leader's (finding 1).

**Card 10**
- Shed restore of a clean detached worktree with a marker: reachable on any resume after a leader dies.
- Refusal at rows other than Finalize: reachable after an operator's `lym loom goto`.
- `detachedStop` dirty or no-marker: reachable, since the shed restore returns nil at Finalize.
- `detachedStop` fail-loud branch for a clean marked worktree: not reachable in production. `newShed` is the only loom shed constructor (`arm.go:303`, `sharedbootstrap.go:485`), so this is a correct wiring guard.

**Card 11**
- `queueBackstop` error releases the lock and fails the verb: reachable through the chmod row. Under an absent upstream it never clears (finding 3).

## Findings

1. **BLOCKING, design.** Cards 8 and 9 (`internal/landingshed#Finalize.leadBatch`, `Finalize.bisectBatch`): when the leader's own chain link is `Conflicted` or `Held`, the outcome is undefined.
   - Card 8 sets `conflict` on any Conflicted member and leaves a Held one out, then lands alone after the rebuild bound "through today's merge and gate". Card 7's `mergeNeedsSession` covers only merge-ins under the lock, not the chain.
   - The state is reachable: card 8 runs `syncParent` after `actOnOutcome` answered none ("after `actOnOutcome` answered none and `syncParent` passed"). `syncParent` can fast-forward the parent to an upstream commit that conflicts with the leader's task head, or that holds its work (`finalize.go:755-766`).
   - A Conflicted leader would then run the conflict session under the lock, which D8's last bullet forbids. Or it sets `conflict` on its own entry mid-call and leaves nothing to act on it.
   - Card 9's "culprit is the leader (k = 0)" assumes chain commit 1 is the leader's. With the leader's link left out, a failing prefix 1 blames another member, and `failAsCulprit`'s base merge no longer matches the tree that was gated.

2. **MEDIUM, consistency.** Card 7, test row "a run whose `PullRequest.Mode` read fails writes no entry and lands alone": the row cannot pass.
   - Under the lock, `pullRequestLanding` makes the same `Mode()` read and halts on its failure (`finalize_pr.go:266-268`, `halt(... LandingStopPrefix+beamModeUnreadableReason(err))`).
   - `finalize.go:385` and `finalize.go:431` say the same: the decision under the lock ends the call on a failed read.
   - A failed `Mode` read writes no entry and halts; it does not land.

3. **MEDIUM, design.** Cards 1, 7 and 11 (`UpstreamLacks` in `internal/loomcli#landingDeps` and `internal/fabriccli#queueBackstop`): the backstop wedges a parent that has no origin ref.
   - Card 7's `UpstreamLacks` is the negation of `IsAncestor(commit, "refs/remotes/origin/"+parent)`. Card 11's is the same against "the branch's upstream ref".
   - `gitrepo.IsAncestor` returns an error for any exit other than 1 (`ancestry.go:28-37`), so an absent ref is an error rather than "lacks".
   - Card 1 keeps the mark on a seam error. Card 7 then fails `Call` on every re-step, and card 11 fails `lym fabric anchor` and `lym fabric pull --hub` every time. Card 11's way forward, "clear the cause", has no cause to clear.
   - The state is reachable: `ParentNoUpstream` is a legal relation (`finalize.go:740`), and a leader that dies after the move leaves the mark.
   - No card defines the absent-ref case, so the two seams cannot be confirmed to behave alike here.

4. **MEDIUM, consistency.** Card 7, row "a backstop `WriteLanding` error ends `Call` with that error and runs no merge": the row contradicts the card's own order.
   - The card orders `Call` as `mergeUnlocked`, then `joinQueue`, then `landUnderLock`, and runs the backstop inside `landUnderLock` after `guard.ensure`. The lock-free merge through the resolver has already run when the backstop fails.
   - "Runs no merge" is false. It can hold only as "no merge under the lock" or "no parent-side merge", which the row should say.

5. **MEDIUM, design.** Card 8's ledger writes in `internal/landingshed#Finalize.landPrefix`, and the backstop writes in cards 7 and 11, skip the push-skip guard.
   - `pushParent` records no landing under `Deps.PushSkipped` (`finalize.go:1045-1053`).
   - Card 8 calls `WriteLedgerLanding` once per prefix entry with no such check, and the backstop's `WriteLanding` writes with none either.
   - Under the push skip, the batch path leaves ledger landings that the run's own path never writes.

Tool calls:
1. `grep` of `internal/fabricengine/mergeguards.go` for the not-synced guard, to compare `MoveCodeTo`'s guards with `Merge`'s. It refuses only a diverged parent, so I raised no finding there.
2. `grep` for `JudgeOnly` across `internal` and for `Mode()` in `internal/landingshed/finalize_pr.go`. This checks whether card 8's judge-only reconcile exists and how the decision under the lock treats a failed mode read.
3. `grep` for `loomrecipe.New(` and `c.newShed()` callers, plus `internal/lymdirs` junction functions. This checks whether every Finalize-driving shed gets card 10's `Restore`, so whether `detachedStop`'s fail-loud branch is reachable.
