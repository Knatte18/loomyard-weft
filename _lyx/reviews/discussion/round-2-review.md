---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Decision 4, Decision 5, acceptance criterion on pairs created by lyx batten run
    summary: A pair forked while another holder's walk runs is missed by that walk, which still stamps fresh, so Decision 4's stamp-gated second walk is a no-op and the new pair keeps the retired key
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 7, mid-merge skip
    summary: Decision 7 triggers the skip on Fabric.Commit's refusal, which fires after reconcile has written, yet promises the worktree is not written at all; the record never states the pre-write merge check that outcome needs
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks, second bullet
    summary: The cost risk names only the first start verb, but a skipped mid-merge worktree keeps the stamp stale, so every start verb and every Preflight re-walks the hub, and lyx batten run walks twice
---

### [MEDIUM] A pair forked during another holder's walk escapes reconcile

**Location:** `_lyx/discussion/decision-record.md` Decision 4 (second walk), Decision 5 (walk order and stamp write), and the acceptance criterion "A pair created by `lyx batten run` after a binary change carries no retired key ... this holds too when the prime was mid-merge and skipped".

**Issue:** Decision 5's holder enumerates pairs once, after the hub-wide reconcile, then commits the prime and every enumerated pair and writes the stamp.
Pair creation takes no lock the walk takes: `internal/fabricengine/add.go` acquires no write lock on the prime (it only ensures the new pair's weft lock dir), and the prime's `Fabric.Commit` holds `.weft/weft.write.lock` only inside `commitBothSides`.
So a pair forked from the prime's records branch after holder B enumerated pairs but before B committed the prime gets the prime's stale config and is not in B's list, and B still stamps fresh.
In Decision 4's mid-merge path this is reachable: `lyx batten run` A skips the mid-merge prime, the merge ends, start verb B starts its walk, A forks the pair, B commits the prime and stamps fresh.
A's second walk then calls the entry point, whose double-check (Decision 5) sees the fresh stamp and does nothing, so the new pair keeps the retired key, contradicting Decision 4's "that walk reconciles and commits the new pair" and the acceptance criterion.
The same window opens for a direct `lyx fabric add` during any walk, which Decision 4's last paragraph covers only "before any start verb ran".
Loom-Preflight's unchanged retired-key refusal still catches the pair, so the failure is loud, but it lands on the manual `lyx config reconcile --apply` route the Goal removes.
The root cause is one: the stamp asserts every pair is reconciled, while pair creation does not serialize with the walk.

**Fix:** Bound the claim by design, not by timing, and state the chosen route in Decision 4 and Decision 5.
Either the second walk reconciles and commits the new pair itself, unconditionally on the stamp (it is the one worktree `lyx batten run` knows may have forked stale config), or the holder re-enumerates pairs before writing the stamp and reconciles any pair it has not seen.
State in Decision 4 what still escapes (a pair made by `lyx fabric add` mid-walk, if the route chosen does not cover it) and that Loom-Preflight's retired-key refusal is the backstop for it.

### [MEDIUM] Decision 7's mid-merge skip promises "not written" but triggers on a post-write refusal

**Location:** `_lyx/discussion/decision-record.md` Decision 7, the paragraph "A worktree whose records side refuses the commit because a merge is in progress (`ErrMergeInProgress`, foreign merge state) is not written at all".

**Issue:** `Fabric.Commit` (`internal/fabricengine/commit.go`) refuses with `*ErrMergeInProgress` or `*ErrForeignMergeState` when it is called, which in the walk's order (Decision 5: `configsync.ReconcileAll`, then the commit) is after the reconcile has written the files.
Read literally, the trigger leaves the reconciled `_lyx/config` files written and uncommitted in a mid-merge worktree, which is the failure Decision 7's own rationale names (uncommitted config stopping `Preflight`'s clean check and a Finalize merge), and in a mid-merge worktree the stray files can also ride into the merge commit.
The outcome "not written at all" needs a merge-state check before reconcile writes, and the record never states one; `foreignMergeStatePresent` is unexported, so the check is not free for the plan writer to assume.
A merge that starts between such a pre-check and the commit is not bounded either.

**Fix:** State in Decision 7 that the walk checks each worktree's merge state (fabric's merge record and foreign git merge state) before reconciling it, and skips the worktree unwritten when either is present.
State what happens when the commit still refuses after the write (a merge started in between): either the written files are restored, or the start verb stops as Decision 9 states, naming the worktree.

### [LOW] The cost risk understates the cost while a worktree stays skipped

**Location:** `_lyx/discussion/decision-record.md` Open risks, "The first start verb after a binary change is slower ...".

**Issue:** Decision 7 leaves the stamp stale while any worktree is mid-merge, and Decisions 3 and 10 run the walk whenever the stamp is stale.
A long landing conflict in the prime or a pair therefore makes every start verb and every run's `Preflight` row repeat the full hub walk under the hub lock, and `lyx batten run` walk twice (Decision 4), until the merge ends.
The risk names only the first start verb.

**Fix:** Extend the risk: while a skipped worktree keeps the stamp stale, every start verb and every `Preflight` repeats the walk, and `lyx batten run` walks twice.

## Focus verdicts

- Decisions 4 and 7, mid-merge prime: the record now states the outcome in all three places (Decision 4's second walk, Decision 7's cross-reference, the acceptance criterion) and states it consistently; the bound fails under the race in F1, and the write ordering in F2.
- Decisions 9 and 10, `Preflight`: Decision 9's bound names `Preflight` explicitly ("every run's `Preflight` row goes `Stuck`"), and Decision 10 states the way forward for both refusals (fix the file, then `lyx loom resume`; `lyx loom resume` once the other reconcile ends).
  Open risks carries no separate entry for it, which is not a defect: the bound is stated once, in Decision 9.
  Not a defect.
- Decision 14, role membership: membership is the named set `roleOpeningStencils` in `contracts/stencils/stencils.go`, which lists `burler-review`, `bouncer-judge`, `treadle-judge`, `friction` (reflection) and `describe` among others; the PATTERN entry and acceptance criterion use the same "every spawned role's opening stencil" wording.
  The scan has one definition to test against.
  Not a defect.
- Decisions 5 and 10, wait bound: Decision 5 states that the holder's per-worktree waits are unbounded, that the hub-lock wait is a waiter's only bound, and that the plan picks it (Q11, open for the plan); Decision 10 states `Preflight`'s hub-lock timeout as `Stuck` with a way forward.
  I checked for a self-deadlock where `Preflight` as holder commits into its own pair: the shed run lock (`internal/shedengine/run.go`, `s.LockPath`) is distinct from fabric's `.weft/weft.write.lock`, which `Fabric.Commit` holds only for the commit, so the driver holds no lock the walk needs.
  Not a defect.
