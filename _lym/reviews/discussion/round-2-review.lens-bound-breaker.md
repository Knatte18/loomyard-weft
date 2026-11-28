```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break every bound, guard and claim the record makes.

**Claims attacked**
- D1 rationale, "a conflict session never holds the lock for a run that merged cleanly": **broken** (finding 1).
- D2, an entry is live by its pid and start time, and pruning comes only after the backstop: held.
- D2, an entry with an outcome stays until its run reads it: held, but it leaks (finding 8).
- D3, a batch of one keeps today's behaviour, fixer included: held for the run itself. Its merge under the lock is covered by finding 1.
- D3 and D6, the member checks and the left-out bound: **broken** on cost (finding 3).
- D4 with the Constraint, `HoldsWork` tree equality reads a squash chain commit as holding its run's work: **broken** once later members of the batch touch the same lines (finding 2).
- D5, the darn scope "diffs the pair's merge base against the chain commit": **broken** as a claim about the code; the scope is wider than stated, which is harmless (finding 7).
- D5, the marker guards the pusher and refusal reads: held, but only as far as the record states it.
- D6, at most `landing.batch_max` rebuilds per lock hold: held as a count.
- D6, "the guarded move lands exactly the gated tree or nothing": held for the head. The dirty, mid-merge and records-commit steps before the move are not covered (finding 4).
- D6 rationale, "the parent is never ahead of its upstream by a commit that is not an owed landing tip": **broken** while the ledger landings are being written (finding 5).
- D6 Bound, a member's records pushed before the ledger write are harmless: held. A hold on a commit origin already has leaves the ceiling at its parent, so nothing more is sent. The `oldestHeldRecords` paraphrase is wrong (finding 6).
- D6 backstop by `lym fabric pull` and `lym fabric anchor`: held. Both go through `acquireBranchLock`, `internal/fabriccli/anchordeps.go:550`, and no other `Acquire` caller exists.
- D7, about log2(n) extra gates, and a batch of four costs two: held per chain. A culprit that is the leader also costs two.
- D7, the leader's merge of the chain base reproduces chain commit 1's tree: held. Both go through git's ort merge with the same merge base.
- D8 `conflict`, front-of-queue retry: no bound on repeated conflicts now that `finalize_lost_races` is gone; this is minor (folded into finding 1).
- D9 defaults and the reading of a value below 1: held.
- Constraint, the idle release keeps a leader's lock across its gates: held. `finalizeKeepsLock` keeps the lock while the status names Finalize running, `internal/loomcli/looplock.go`.

**Findings**

1. **MEDIUM, design.** `_lym/discussion/decision-record.md:65` (D3) and `:158` (D8 `failed`).
   - Summary: the batch-of-one path, the land-alone path after the rebuild cap (`:114`) and the `failed` own path all run "today's merge and gate under the lock". Only the `conflict` bullet (`:161`) frees the lock before a conflicting merge. A run whose lock-free merge was clean but whose parent moved meanwhile, which is the common case on a busy parent, therefore runs its conflict session while holding the lock. That contradicts D1's rationale (`:39`).
   - Evidence: `internal/landingshed/finalize.go:528` (`mergeAndGate` under the lock) and `:1404` (`resolver.Resolve`, which drives the conflict session). `freeLock` is called only by `runFixer` and `judgeFixerChange` (`:1216`, `:1303`).
   - Fix: apply the `:161` rule, which frees the lock and runs the lock-free merge first, to every merge under the lock.

2. **MEDIUM, design.** `_lym/discussion/decision-record.md:202` (Constraints, PATTERN-lym-files) and `:209` (`HoldsWork`).
   - Summary: `HoldsWork` needs the merge of the task head into the parent to be clean and to leave the parent's tree unchanged. Say member i adds a line and a later member j of the same batch changes that line. Then `merge-tree(parent, head_i)` conflicts, or with a revert it re-adds the line, and `ParentHoldsTaskWork` reads false for a member that has landed.
   - With its outcome lost (the queue is a cache), that member leads a batch. D4's holds-work skip misses it too, so it lands a second time: as `conflict`, or as a second commit that re-applies its reverted change.
   - D8's "writes its ledger landing… its landing commit" also has no landing commit to name once the entry is gone.
   - Evidence: `internal/fabricengine/parenttips.go:225-247` (a conflicted merge holds nothing; the result must equal the parent's tree) and `internal/fabricengine/replay.go:336-345`. The `Lym-Landing` trailer is already read on the parent by `landingCommitsOn` (`internal/fabricengine/landing.go:304`) and by `unrecordedBlockers` (`internal/runpusher/parent.go:461`).
   - Fix: judge "landed" by the trailer naming the slug on the parent, which also gives the landing commit, not by tree equality alone.

3. **MEDIUM, design.** `_lym/discussion/decision-record.md:62-64` (D3) with `:103-111` (D6).
   - Summary: D3's selection skips only paused, moved-tip and outcome-carrying entries. The owner, reconcile, land-check and resolver checks run only when a passing prefix is landed, after the gate. A member left out by them keeps no outcome and stays live in the lock wait, so every later leader takes it again.
   - Each such leader pays one gate plus one rebuild gate for it until the member reaches the lock itself. A member owned elsewhere or a GitHub-bound member costs a gate per leader. D7's "about log2(n)" cost does not cover this.
   - Evidence: `_lym/discussion/decision-record.md:110-111` (left out with no outcome) and `:62-63` (selection criteria).
   - Fix: run the read-only member checks before the first chain build too, keeping the re-read before the move.

4. **MEDIUM, design.** `_lym/discussion/decision-record.md:117` (D6 move).
   - Summary: today `syncParent`'s dirty and mid-merge probe and `commitParentRecords` run right before the parent-side merge. D6 puts the batch gate and up to `batch_max` rebuilds and bisections, which can be hours, between `syncParent` and the move. It names no commit of the parent's run records and no dirty or mid-merge refusal path at the move.
   - The parent pair can be dirtied in that window: sibling runs' uncommitted status records, and the operator's work in the prime.
   - Evidence: `internal/landingshed/finalize.go:536-540` (step 2b exists because siblings leave status records uncommitted) and `:710-723`.
   - Fix: have D6 commit the parent's records and re-probe dirt and mid-merge right before the move. Name the outcome of a refused move: the mark is removed with nothing written, and the batch lands nothing.

5. **LOW, design.** `_lym/discussion/decision-record.md:118` and `:127` (D6).
   - Summary: the leader writes one ledger landing per entry, and `runpusher.RecordLanding` restarts the parent pusher on every call. A pusher woken after entry 1 sees the landing commits of entries 2..k on the parent, unrecorded, of runs still at Finalize. `unrecordedBlockers` turns them into a held refusal.
   - That breaks the rationale's "never ahead by a commit that is not an owed landing tip" for the window. A refused parent push halts a GitHub landing through `parentPushBlocks`.
   - Evidence: `internal/runpusher/parent.go:458-490` and `:507`, `internal/runpusher/parent.go:719-737`, and `internal/landingshed/finalize.go:606`.
   - Fix: write every entry's ledger landing in one `pushledger.Update`, then wake the pusher once.

6. **LOW, consistency.** `_lym/discussion/decision-record.md:116` (D6).
   - Summary: "the oldest one origin lacks, as `oldestHeldRecords` picks it today" misstates the code. `oldestHeldRecords` returns a standing ledger landing's records commit when a given commit descends from it and origin lacks it; it never searches for the oldest unpushed commit. The function that does that is `heldByRefOrLanding`'s `oldestUnpushed`.
   - Evidence: `internal/loomcli/landingdeps.go:982-1012` and `internal/fabricengine/landing.go:312`.

7. **LOW, consistency.** `_lym/discussion/decision-record.md:97` (D5 Bound).
   - Summary: the darn scope read takes its base from `taskMergeBase`, which reads the merge record's `MergedTip` first, not the pair's merge base. A leader whose merged tip is older than the chain base therefore scopes to the members' trees plus the trees of parent commits landed since its merge. That is a superset, harmless, but the stated mechanism is wrong.
   - Evidence: `internal/loomcli/subjects.go:127-138` and `internal/loomcli/darnwiring.go:89,104`.

8. **LOW, design.** `_lym/discussion/decision-record.md:54` (D2).
   - Summary: an entry with an outcome is removed only by its own run, resumed. A run cancelled or torn down after a `landed` or `failed` outcome never resumes, so its entry and outcome stay in the queue directory for good. No bound or pruner is named.
   - Evidence: `_lym/discussion/decision-record.md:50,54`.

Tool calls:
1. Grepped `internal/runpusher` for the unrecorded-landing refusal and found the callers of `RunCommitsAhead` and `TipsOnOrigin`, to test D6's never-ahead claim and D5's detached readers one hop past the manifest.
2. Read `unrecordedBlockers` and `pushParent` (`internal/runpusher/parent.go:455-540`) and the `TipsOnOrigin` callers in `beamcli` and `battencli`, to build the ledger-window counterexample; the manifest had only `RecordLanding`.
3. Read the dobby's lock-store use, anchor's lock section and `taskMergeBase`, to check the remaining lock takers and the base of D5's darn scope claim; the manifest stopped at `darnTreeScope`.
