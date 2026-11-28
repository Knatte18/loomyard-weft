```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end against the code in context.

**Decision walkthroughs**
- D1: a run with no PR need merges P lock-free and gates nothing, commits its records, writes its entry and waits in `ensure`. Purpose met.
- D2: the entry is written at join and kept through a halt. A Stuck process is dead, so its entry reads as not live and is skipped. A rejoin rewrites it. Purpose met.
- D3: a leader with no outcome syncs P and takes [L, A, B]. A batch of one runs `mergeAndGate`. Purpose met.
- D4: squash chain over P' for heads T_A and T_B, each merged at its own join tip. merge-tree's base is P_x, and each chain tree is its prefix's tree. Purpose met.
- D5: the leader is detached at c2. `darnTreeScope` takes the merge base of HEAD=c2 with the parent, which is B, and diffs B..c2 over every member's paths, so the Bound holds. The restore and marker steps check out. Purpose met.
- D6: prefix [L, A] lands, the mark is written, the move is made, ledger landings and outcomes follow. Purpose met for members. **Not met** for the leader's own tip (finding 2). The records-commit wording is off (finding 3).
- D7: batch of 4, B fails. Gates run on prefixes 4, 2 and 3; prefix 2 lands, B gets `failed`, C stays with no outcome. With the leader as culprit, prefixes 4, 2 and 1 fail; merging B into the leader gives c1's tree, and the fixer runs on the prefix-1 log. Purpose met.
- D8: member A is landed in c2 and wakes on the lock. **Purpose missed** (finding 1).
- D8 `failed` / `conflict`: B gates alone under the lock, its fixer runs lock-free, and it lands alone. The conflict session runs lock-free. Purpose met.
- D9, D10: a leader sent to a PR lands alone, and a member sent to a PR is left out. Purpose met.
- D11, D12: review-only checks. Nothing to walk.

**Findings**

1. BLOCKING, design. `_lym/discussion/decision-record.md:152-153` (D8's opening paragraph).
   - Summary: every batch-landed member pays its own full landing gate under the parent lock, which undoes the `landed` bullet and the record's Goal.
   - Walkthrough: member A's squash landing c2 is on the parent, so `ParentHoldsTaskWork` reads true when A takes the lock. D8 then sends A to "today's already-landed path" before it reads the outcome.
   - Today's already-landed path is `landUnderLock` with `held` true. It skips the owner checks, the sync and the tip compare, then still runs `mergeAndGate`:
     - it merges the parent, which now carries L's commit too, into A's worktree;
     - it runs the gate in A's own VerifyDir, which has never verified that tree, so the full tiers 0 to 2 run;
     - on a failure it runs A's fixer, though A's work already landed;
     - `pushParent` then calls `LandLocally(parent HeadSHA)`, and `runpusher.RecordLanding` replaces the ledger landing the leader wrote with the parent head as A's tip.
   - The same holds for a dead leader resumed after its move.
   - The D6/D8 criterion "a run woken with outcome `landed` runs no merge and no gate" contradicts the held-first ordering.
   - Evidence: `internal/landingshed/finalize.go:473-529` (`held` read, `syncParent` and `landCheck` skipped, `mergeAndGate` unconditional), `:280-285` (already-landed path), `:1034-1054` (`LandLocally` of HeadSHA), `internal/runpusher/parent.go:719-725` (replace by run).
   - Fix: give a run whose work the parent holds the no-merge, no-gate path: write its ledger landing with its own landing commit if missing, do its remaining records, end. Don't use today's merge-and-gate path for it, and don't call `LandLocally` with the parent head while a landing of the run stands.

2. MEDIUM, design. `_lym/discussion/decision-record.md:103` (D6, the leader's checks).
   - Summary: the leader's own task-branch tip can move after its chain was gated, and D6 never re-reads it.
   - Walkthrough: under the lock the leader runs "today's owner checks for its own run" only when it lands a prefix, after the chain was built and gated. Their run-scope reconcile can fast-forward the task branch. D6's pre-move re-read covers members' tips only.
   - Result: the leader's old head T_L lands in c1 and gets `landed`. The leader ends Done, and the fast-forwarded commits never land.
   - Today's flow runs the owner checks before `mergeAndGate`, so a fast-forward gets gated.
   - Evidence: `internal/landingshed/finalize.go:469-486` (comment: "The owner checks' run-scope reconcile can fast-forward the task branch", with `held` re-read after them).
   - Fix: run the leader's owner checks under the lock before it takes its batch, and add the leader's own task-branch tip to D6's pre-move re-read.

3. LOW, consistency. `_lym/discussion/decision-record.md:116` (D6, the member's records commit).
   - Summary: "the oldest one origin lacks, as `oldestHeldRecords` picks it today" misstates that function.
   - `oldestHeldRecords` returns an earlier records commit only if a standing ledger landing of the run holds it and origin lacks it. Otherwise it returns the commit it is handed. A member has no standing landing before the leader's write, so the result is whatever the leader hands it, and the record never says what that is.
   - The real oldest-unpushed picker is `heldByRefOrLanding`'s `oldestUnpushed`.
   - Evidence: `internal/loomcli/landingdeps.go:982-1013`, `internal/fabricengine/landing.go:275-320`.
   - Fix: name the commit the leader hands for each member, for example the member's records head read from its records worktree at the move, falling back to the join-time head. Cite the picker that computes "oldest origin lacks".

Tool calls: none.
