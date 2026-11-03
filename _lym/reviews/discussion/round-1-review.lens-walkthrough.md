```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**List — decision, case, purpose met or not**
- D1, case A: resume at Run-Shed after the task worktree dir was deleted by hand, with both branches still local and on origin. **Not met.** In step 1 the reconcile gets no entry for the slug, and step 3 refuses a beam project's pair because of its anchor.
- D1, case B: resume after a plain `lym fabric remove`, where the weft tip on origin is covered by an archive tag. **Met.** `Add` returns the not-live sentinel and the case goes to D2.
- D2, case A: permanent answer at Run-Shed. **Met.** `WriteFailedStop` is not self-Busy, because the pre-run holds no run lock (arm.go:899-904 probes the lock and releases it).
- D2, case B: permanent answer at Land. **Not met.** The way forward re-runs an `Add` that has already been refused, or forks a fresh pair.
- D3, case A: Finalize's "origin's parent holds the work" stop, then `lym fabric remove`, then `lym batten cancel`, at Run-Shed. **Met.**
- D3, case B: a stop naming `lym fabric remove` while batten stands at Land. **Not met.** The cancel clears the record of a landed entry.
- D4, case A: normal removal while a detached push is still pending, so origin is at O, an ancestor of the local tip L. **Met for D4 alone**, but its output does not feed D6 (see D6).
- D4, case B: origin O diverged from L. **Partly met.** Tagging O needs O's objects locally, and step 1 only "reads" O.
- D5, case: 2026-11-02 case, a commit at the remote branch tip. **Met.** For a commit that is an ancestor of the remote tip, it is **not met**.
- D6, case A: the D4 case A removal, then the leased delete. **Not met.** The lease input is ambiguous, and read literally the remover breaks its own lease.
- D6, case B: the cleanup sweep of an orphan weft branch. **Not met.** No origin tip is read, so the lease has no input.
- D7, case: `Add` while the former pair's detached push holds the lock shared, then `Add` returns and the push is spawned. **Met.**

**Findings**

1. **BLOCKING · design · D1 step 1 (and acceptance criteria D1 #3 and #4)**
   - "Judges the run's entry against origin as it does today" gets no entry when the pair worktree is absent.
   - `classifyEntries` judges only `recorded`, which comes from `pairs`. `pairs` is filled only from warp worktrees present in scope (originreconcile.go:640-652, 716, 749, 818-843).
   - With the worktree gone, the slug yields no `EntryOutcome`, so `resumeRefusal` returns nil (resumereconcile.go:113-131). The owner, ended and stopped verdicts that D1's bound relies on silently pass.
   - The record must state that the run scope classifies the slug's entry without a pair worktree, not "as today".

2. **BLOCKING · design · D1 step 3 ("calls `Add` with the run's parent and anchor")**
   - Passing the anchor makes every re-adoption of a beam project's live pair refuse.
   - Worktree-Create passes `Anchor: anchorFromSeed(...)` for a project (wire.go:962-973). `Add` refuses any non-empty `opts.Anchor` when the weft is live (add.go:398-401).
   - D2 classes that refusal as permanent, so a recoverable beam pair is failed instead of re-adopted.
   - For a task with no named parent, `Parent: ""` also triggers `Add`'s source-dirt check on the prime (add.go:239-246). A dirty prime then gives `ErrSourceDirty`, which D2 also classes as permanent.
   - Adopting a live pair needs neither the anchor nor a clean source, and the record does not say so.

3. **BLOCKING · design · D2, Land way forward**
   - `lym fabric add <slug> --adopt-local` then `lym batten run <slug>` cannot recover the cases D2 sends it.
   - On the not-live sentinel, the archived weft is replaced. A hand add without the adopt-only option forks a fresh weft from the parent's records branch (add.go:175-177, remoteleftover.go:100-103), so the child's run records are gone before Land resumes.
   - On a records-worktree-present refusal, the same `Add` refuses again (add.go:328-331).
   - The status is also `failed` by then, and the record does not say how `lym batten run` treats a `failed` status at Land.

4. **MEDIUM · design · D3 rule and rationale**
   - The blanket "name `lym batten cancel` after `lym fabric remove`" also reaches stops where batten stands at Land.
   - The rationale misreads `childReachedParent`. With the child status gone, it returns true only at Worktree-Teardown or when past the merge; at Land it returns false (cancel.go:691-698).
   - So at Land the appended cancel clears the record and claim of a landed entry, the result D2 forbids.

5. **MEDIUM · design · D4 step 2 → D6 lease**
   - The tip D6 leases on is not the tip D4 leaves on origin.
   - D4 step 1 reads origin tip O, and step 2 pushes L, moving origin to L. D6 leases on "the origin tip the deleter read … the tip D4 archived".
   - Read as O, every removal whose detached push was pending loses its own lease. It keeps the weft branch and wrongly reports "origin moved during the removal".
   - Read as L, the lease is wrong when the branch push failed. The record must name the lease as origin's tip after D4's push.

6. **MEDIUM · design · D6 for the cleanup sweep**
   - The sweep has no origin tip to lease on.
   - `Cleanup` archives the local tip only (cleanup.go:245, archive.go:84, 140-146) and deletes on origin unleased (cleanup.go:257-258, 368-378). It never reads origin's tip, and D4 does not run there.
   - "The tip D4 archived" therefore has no value on this path. Leasing on the local tip is the rejected alternative, which keeps an uncovered tip that `Add` reads as live.

7. **LOW · design · D4 step 3**
   - The archive tag at a diverged origin tip O needs O's objects locally, and step 1 only reads O.
   - `tagAndPushArchiveTip` runs `git tag <tag> <tip>` locally (archive.go:120). `resolveArchiveTip` fetches only when there is no local branch (archive.go:140-172).
   - D4 should state that O is fetched before it is tagged.

8. **LOW · design · D5**
   - Comparing `git ls-remote <remote> <ref>` with the commit the stop names reads only branch tips.
   - A commit the remote holds as an ancestor of a tip still compares unequal. The stated purpose, "a commit the remote holds is never read as absent", is met only when the commit is the tip.

**Tool calls**
1. grep of originreconcile.go entry classification, and the battencli/battenshed `Add` call sites: needed how the run scope derives entries, and how Worktree-Create calls `Add`.
2. grep of battencli for run-lock and `preflightRunOwner` order: needed whether `WriteFailedStop` would be self-Busy in the pre-run.
3. Read of wire.go:950-995, arm.go:715-760 and 890-915, and originreconcile.go:818-834: needed the `Add` options Worktree-Create passes, the pre-run order, and `classifyEntries`.
4. grep of dobbyengine for failed-state handling and `recordedPairs`: needed whether dobby stops on `failed`. Nothing conclusive came back, and no finding is drawn from it.
