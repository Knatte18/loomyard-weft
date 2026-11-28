---
verdict: BLOCKING
findings:
  - id: F19
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:152
    summary: "D8 sends every run whose work the parent holds to today's already-landed path whatever its outcome says, and a member the leader landed always reads as held (squash tree equality), so the `landed` bullet is not reached; today's held path still runs `mergeAndGate` under the lock (internal/landingshed/finalize.go:473-529), merging the parent into the member's worktree and running the full gate in that worktree's own verified-tree record, so each batched member pays its own landing gate again, against the Goal and the D6, D8 criterion that a `landed` run runs no merge and no gate."
    fix: "In D8's opening paragraph, replace 'and takes today's already-landed path, whatever its outcome says' with a queued-run held path that merges nothing and gates nothing: the run judges the held owner checks as today (`ownerChecks` with held, which reads only the reconcile's ended and stopped verdicts) and `handedOverStop`, writes a ledger landing with its own landing commit only when no ledger landing of the run stands and the parent's upstream lacks that commit, does its remaining landing records (as F20 fixes them), removes its entry and ends Finalize. Its own landing commit is the entry's `landed` commit or, with no entry, the commit on the parent whose `Lym-Landing` trailer names its slug. State that this path never runs the merge of the parent into the task worktree or the landing gate. Make the `landed` bullet the same path, so the two no longer differ. Reword the D8 criterion (decision-record.md:244) to: a run whose work the parent holds, whatever its entry says, runs no merge and no gate, writes no landing commit and ends Finalize. Tier 1, `internal/landingshed`."
  - id: F20
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:121
    summary: "D6 has the leader, and D8 a woken member, do their 'remaining landing records' as today, but today's `pushParent` calls `Deps.LandLocally` with the parent's head and `recordLanding` calls `Deps.RecordLanding` with it (internal/landingshed/finalize.go:964-970, 1048-1053); `runpusher.RecordLanding` replaces the run's ledger landing by slug (internal/runpusher/parent.go:719-725), so the leader's tip becomes the prefix's last commit, its own landing commit is then past upstream with no ledger tip, `fabricengine.AheadByOnly` reads false (internal/fabricengine/landing.go:207-218) and the next Finalize's `syncParent` stops Stuck on a parent ahead of its upstream, the state D6's rationale says never arises; the dobby's landed-commit record also names the wrong commit."
    fix: "In D6, after 'The leader then does its own run's remaining landing records', state that a batched run's remaining landing records take its own landing commit, never the parent's head: `Deps.RecordLanding` gets that commit and the empty-diff check measured from its first parent, and `Deps.LandLocally` is not called while a ledger landing of the run stands, so the ledger landing the leader wrote at the move is kept, not replaced. State the same in D8 for a woken member and in the held path F19 adds. In Constraints, extend the `pushledger.Landing` line to say the leader's own run is recorded the same way as a member. Add an acceptance criterion: after a batch of three lands, the push ledger holds one landing per run whose tip is that run's own landing commit, the dobby's landed-commit record of each run names its own landing commit, and the next run to take the lock reads the parent as ahead by owed tips alone. Tier 1, `internal/landingshed`."
prior:
  - id: F1
    status: resolved
    answer: "D6 now judges each member's reconcile verdict, owner read, land check, hand-over read and resolver as reads, and `ReconcileOrigin`'s judge-only scope makes the reconcile read possible; Constraints and the criterion at decision-record.md:235 carry it."
  - id: F2
    status: resolved
    answer: "D1 keeps only `needsGitHubOnDisk` before the lock, D6 and D10 run the resolver per member under the lock, D2 names the seams the entry's pair path builds, and decision-record.md:248 tests it."
  - id: F3
    status: resolved
    answer: "D3, D4 and D8 read held work through the parent, the backstop sets `landed`, and the Constraint names the queue as a cache; the held path's own defect is raised as F19."
  - id: F4
    status: resolved
    answer: "D6 adds the move mark, read before any pruning; the fixer has non-run lock takers run the backstop rather than leave the mark, which also closes the gap."
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
    answer: "Accepted as fixed differently: the marker-based hold-off stands in for a pair push lock that does not exist."
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
---

### [BLOCKING] A landed member goes through today's merge and gate again

**Location:** `_lym/discussion/decision-record.md:152`

**Issue:** The F3 fix put the held-work test before the outcome bullets, and it routes a held run to "today's already-landed path".
That path is `landUnderLock` with `held` true.
It skips the owner checks' full judgment, the land check and the sync, but still runs `mergeAndGate` (`internal/landingshed/finalize.go:526-529`).
That step merges the parent into the task worktree and gates it.
`VerifyDir` is one per worktree, so the leader's pass on the chain commit does not let the member skip the gate.
The leader landed the member with a squash or a true merge, so `HoldsWork` reads the member as held (`internal/fabricengine/parenttips.go:225-247`).
Every batched member therefore takes this path when it takes the lock.
Each one pays a full landing gate under the lock, which undoes the Goal and contradicts the D6, D8 criterion at line 239.
The interactions lens, the walkthrough lens and the handler each found this independently.

### [BLOCKING] A batched run's remaining records overwrite its own ledger tip with the parent head

**Location:** `_lym/discussion/decision-record.md:121`

**Issue:** D6 rebinds only the landed size and the empty-landing check to each run's own landing commit.
It leaves the leader's, and a woken member's, "remaining landing records" on today's `pushParent` and `recordLanding`.
Both read `parentHandle.HeadSHA()` as the run's landing.
`runpusher.RecordLanding` replaces the run's ledger landing by slug.
Take a prefix of leader L and member A, with landing commits c1 and c2.
Once the leader records, its tip moves from c1 to c2, and c1 sits past upstream as no ledger tip.
`AheadByOnly` requires every commit past upstream to be a tip, so `ParentAheadByOwed` reads false.
The next run's `syncParent` is then Stuck "ahead of its upstream" until the parent's push lands.
The dobby's landed-commit record names c2 for L as well.

## Not raised: below the incremental bar

This round is incremental, so it raises new findings only at BLOCKING.
The forks also reported the following.
They are real or plausible, and the parent or a later round may want them:

- Bound-breaker 1 and interactions F-b (MEDIUM): a batch of one, the land-alone fallback after the rebuild cap and the `failed` own path all merge under the lock through the resolver. A parent that moved since the lock-free merge then runs a conflict session with the lock held, against D1's rationale. D8's `conflict` retry loop has no bound.
- Walkthrough 2 (MEDIUM): the leader runs its own owner checks only after the gate, and their reconcile can fast-forward its task branch. D6's pre-move re-read covers members' tips only, so a fast-forwarded leader lands its old head and ends Done without the new commits.
- Interactions F-c and bound-breaker 3 (MEDIUM): member owner, reconcile, land-check and resolver reads run only after the gate. A left-out member keeps no outcome, so every later leader takes it again and pays a gate for it.
- Interactions F-d (MEDIUM): after a bisection, a rebuild for a left-out member does not say whether it covers the passing prefix only, or when the culprit's `failed` is written.
- Interactions F-e (MEDIUM): a leader whose own entry conflicts in the chain has no stated path.
- Bound-breaker 2 (MEDIUM): `HoldsWork` reads a landed member as not held once a later member of its batch rewrites the same lines. With the queue lost, it can be landed again. The `Lym-Landing` trailer is a sounder test.
- Bound-breaker 4 (MEDIUM): between the sync and the move, the parent's run records are not committed again and no dirt or mid-merge probe runs.
- Bound-breaker 5 (LOW): ledger landings written one by one wake the parent pusher while later landing commits are still unrecorded.
- Walkthrough 3 and bound-breaker 6 (LOW): D6 misdescribes `oldestHeldRecords`, which never searches for the oldest commit origin lacks.
- Existing-paths 2 (LOW): `TipsOnOrigin` and `RunCommitsAhead` misread a detached leader as not level.
- Existing-paths 3 and bound-breaker 8 (LOW): no path removes an entry with an outcome whose run is torn down.
- Bound-breaker 7 (LOW): the darn scope base comes from the merge record's merged tip, which gives a harmless superset.
- Interactions F-f (LOW): the D3 criterion's count of `landing.batch_max` reads as excluding the leader.

## Rejected

- Fabric verbs that run the D6 backstop and write ledger landings, as a departure from the declared `PATTERN-landing`: the parent move itself stays in a run's Finalize under the lock. The backstop only records landings already made there, and D12's background change names who writes each ledger landing.
