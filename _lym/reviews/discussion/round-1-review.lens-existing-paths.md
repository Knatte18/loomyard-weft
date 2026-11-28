```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced every state, setting and invariant each decision changes through the code that already runs.

**Paths traced, per decision**

- D1 lock-free merge, then wait in `parentguard.ensure`: survives.
- D1 removal of the lost-race loop: `Config.FinalizeLostRaces` and `LoadConfig`'s check that it is at least 0 go with the key. Survives.
- D1/D8 the run's own under-lock checks: `ownerChecks`, `landCheck`, `pullRequestLanding`, `handedOverStop` and `parentHoldsWork` still run for the leader. A batched member gets none of them. Breaks (F1).
- D2 entry liveness by pid and start time, in the same form as `parentlock.ProcessAlive`: survives.
- D3 `syncParent`'s check that the parent is ahead only by owed tips (`parentAheadByOwed` and `AheadByOnly`) against intermediate chain commits: each one is a ledger tip, so it survives.
- D5 the task pair's background pusher, `readLocalBranch` and `pushPlainWithRetry`, on a detached HEAD: breaks (F3).
- D5 `RefusedPushStands`, run through `runpusher/targetreads.go`, on a detached HEAD: breaks (F3).
- D5 `TipsOnOrigin` in dobby and batten teardown: it runs only after landing, once the worktree is restored. Survives.
- D5/D7 the verified-tree record (`verifytree.runAndRecord`) across chain gates and the leader's later gate on its own path: misread (F5).
- D5 idle release (`finalizeKeepsLock`): the leader's status reads Finalize running, so the lock is kept. Survives.
- D6 `recheckLock` and `stillHeld` before the move: survives.
- D6 the push-ledger write (`recordHeldLanding`, `oldestHeldRecords`) and the pair pusher's ceiling (`recordsPushCeiling`) between join and move: breaks the stated bound (F6).
- D6 `RecordLanding` and `RecordLandedSize`, which measure from the process-local `parentHeadMerged`: D6 moves the base to the first parent. Survives.
- D8 `parentHoldsWork` and today's already-landed path, against an entry with no outcome after the backstop: breaks (F2).
- D8 `conflict` outcome running `mergeParent`, then `resolver.Resolve` under the lock: breaks D1's rationale (F4).
- D9 the retired key in a hub default layer, through `warnUnknownKeys`: the warning is the unknown-key warning (F7).

**Findings**

**F1 — BLOCKING · design · `decision-record.md` D6/D3: a batched member skips every check today's Finalize runs on that run under the lock.**
- Under the lock, `landUnderLock` runs these checks for the run itself (`finalize.go:458-491`):
  - `pullRequestLanding`;
  - `ownerChecks`, the run-scope reconcile: ended, stopped, owned elsewhere and hand-over (`finalize_owner.go:40-70`);
  - `landCheck` for a beam's members (`finalize_land.go:35-47`).
- Before recording the landing, `pushParent` runs `handedOverStop` (`finalize.go:1035`).
- D6 has the leader run "today's owner checks", which judge the leader alone. A member whose entry another machine took over, or whose entry origin ended or stopped while it waited, still lands on the local parent.
- That breaks `PATTERN-owner-machine`: a landing proceeds only when the run's entry does not read as owned by another machine.
- A pawl member's beam blockers are never read either.
- Fix: before the move, the leader runs each member's reconcile verdict, owner read and land check. A member that fails one is dropped from the chain and rebuilt, or gets an outcome that sends it down its own path. Add a criterion for it.

**F2 — MEDIUM · design · D6 backstop and D8: a run whose work the parent already holds but whose entry has no outcome leads a batch and lands its work again.**
- The backstop writes missing ledger landings but sets no outcome.
- D8 dispatches only on the outcome, so the member leads a batch (D3) that holds its own entry.
- D4's `commit-tree` writes a landing commit for an entry whose head the chain base already holds. Today `Fabric.Merge` returns up to date for that case (`merge.go:444-450`). Today's `held` check routes such a run to the already-landed path (`finalize.go:371-377`, `473-486`; `deps.go:147-154`).
- Fix: the backstop sets `landed` on each entry it covers. D3/D4 also leave out any entry whose head the base already holds, and send that run down the already-landed path.

**F3 — MEDIUM · design · D5: processes outside the batch read the detached worktree's HEAD, so they fail or misread while the leader gates.**
- The pair pusher's `readLocalBranch` reads HEAD through `CurrentSHA`. On a detached HEAD, `UpstreamSHA` answers `ErrNoUpstream`, so the code side reads as unpushed (`pusherreads.go:37-59,155`; `blobread.go:376-377`).
- The push itself resolves `CurrentBranch`, which errors with "HEAD is detached" (`push.go:146-154`; `gitrepo.go:314-315`).
- `RefusedPushStands` errors the same way (`pusherreads.go:170-178`, called from `runpusher/targetreads.go:115`).
- D5's rule that reads resolve the branch ref covers only "the batch path".
- Fix: D5 says the pair's pusher and its refusal read stand still while the leader is detached, for example under the pair push lock held across the detach. Otherwise those readers resolve the task branch ref.

**F4 — MEDIUM · design · D8 `conflict`: the conflict session runs while the run holds the parent lock.**
- The run's own path is `mergeAndGate`, then `mergeParent`, then `resolver.Resolve` (`finalize.go:1091`, `1404`).
- `freeLock` runs only before a fixer or a fix review (`finalize.go:1216`, `1303`).
- `finalizeKeepsLock` keeps the lock past the idle bound while Finalize is running (`looplock.go:202`).
- So a conflicted member holds the lock through a conflict session of up to `conflict_timeout_min` (60 minutes). D1's rationale says that must never happen.
- Fix: a `conflict` run merges with no lock held, then takes the lock and lands alone.

**F5 — LOW · design · D7: when the leader is the culprit, its own path gates again a tree the batch already found failing.**
- Prefix 1 is the leader's merge onto the chain base. Its gate failed, after the rerun.
- `runAndRecord` records passes only (`verify.go:461-470`, `487`). Each chain pass also replaces the leader's entry for the command (`verify.go:338`).
- So "today's merge, gate and fixer alone" runs one more full gate, tiers 0 to 2, before the fixer. That gate is not counted in "about log2(n) extra gates".
- Fix: the leader-culprit goes straight to its fixer on the prefix-1 failure log, or D7 counts the extra gate.

**F6 — MEDIUM · consistency · D6 Bound: nothing holds a member's records from its join on.**
- The pair pusher's ceiling exists only once a ledger landing names the run (`pushanchored.go:104-112`; `ledger.go:239-261`).
- Between join and move there is none, so the member's pusher may push the join-time records commit and later ones.
- The leader then writes a ledger landing whose records commit origin already holds. `oldestHeldRecords`, which today's path uses (`landingdeps.go:982-1011`), avoids exactly that and is not on the leader's path.
- Fix: either D6 names the mechanism that holds from join, or the bound is cut and the ledger takes the member's newest records commit that origin lacks at the move.

**F7 — LOW · consistency · D9 and its criterion: "warned of as a retired key" is not what the code does.**
- In a default or worktree layer, a key listed in `RetiredKeys` is an unknown key. `warnUnknownKeys` warns about it as unknown and names the reconcile (`configengine/config.go:324-326`; `doc.go:71-75`).
- `RetiredKeys` changes only the local and machine layers' refusal.
- Fix: D9 and its criterion name the unknown-key warning, and say whether the key joins `RetiredKeys`.

**Tool calls**

1. `readLocalBranch` and the lock-taking callers: to see how the pusher reads a detached worktree, and who else takes the parent lock.
2. `UpstreamSHA`, the `readLocalBranch` callers and `Push`: to see how the pusher's reads and the push behave on a detached HEAD.
3. The `pusherreads.go` code-side reads and `pushWithRebaseRetry`: to see which push paths read HEAD.
4. `pushPlainWithRetry`, `CurrentBranch`, and the callers of `TipsOnOrigin` and `RefusedPushStands`: to see how the push resolves its branch and who reads the pair's tips.
5. The dobby and batten callers of `TipsOnOrigin`: to judge whether they run while the worktree is detached.
