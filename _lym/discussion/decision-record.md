## Goal

Runs ready to land on one parent branch stop paying one landing gate each.
Runs at Finalize join a landing queue for their parent branch, and the parent lock's holder gates and lands a batch of them with one gate run.
A failed batch is bisected: the runs before the culprit land, and the culprit goes back to its own run's fixer.
The batch is gated exactly as it lands: the parent's new tip is the commit the gate ran on.

## Scope

In:

- A landing queue per parent branch, written by Finalize, in a new package `internal/landingqueue` (new).
- Finalize's flow: the lock-free gate and the lost-race loop are replaced by the queue; a batch of one keeps today's merge and gate under the lock.
- Fabric verbs that build a chain of landing commits without a worktree, check a task worktree out at a commit and back, and move a parent pair to a chain tip only from the chain's base.
- Bisection of a failed batch, the batch's ledger landings, and the wake-up paths of a queued run.
- `landing.batch_max` added; `landing.finalize_lost_races` removed.
- `internal/landingshed/doc.go`, the glossary, `PATTERN-landing` and its background file.

Out:

- Pull-request landings (D10).
- Gating a batch's prefixes side by side, which the entry's Idea offers as a step that "may" follow: it multiplies the gate-slot contention the entry reports.
- Gate slot priority, which the note `lock-holder-gates-first` owns.
- Landings across machines: a parent lands only on its owner machine, so one queue per parent per hub suffices.

## Decisions

### D1 — Finalize joins the queue instead of gating with no lock

A run at Finalize runs its preamble and owner checks, then merges the parent's tip into its task worktree with no lock held, through the resolver, as today.
It runs no gate there.
It reads on disk, with no lock and no GitHub call, whether its landing is known to need GitHub (`needsGitHubOnDisk`, as today before the lock);
a run known to need it lands alone (D10).
Any other run commits its records, writes its queue entry (D2) and waits for the parent lock through `parentlock.Store`'s ticket queue, as today.
Whether it lands through a pull request is decided under the lock (D6, D10).
Under the lock it acts on its entry's outcome (D8).
The lock-free gate, the lost-race comparison and the lost-race count are removed.
Rationale: every landing on a busy parent made each waiting run's lock-free gate stale, and those gates competed with the lock holder for gate slots;
the merge stays lock-free so a conflict session never holds the lock for a run that merged cleanly.

### D2 — The queue is one entry file per run under the hub's `.lym`

`internal/landingqueue` (new) owns a directory per parent branch, named by `lymdirs` and keyed by the branch's hex like `internal/parentlock`'s records, holding one JSON entry per queued run, written through `internal/state` under a `lock` file lock.
An entry carries the run's slug, pair path, run id, pid and start time, its task head, its records head at join, the landing message from its change description, whether its landing squashes or true-merges, its join time, and an outcome:
none, `landed` with its landing commit, `conflict`, or `failed` with the chain commit it failed on.
The records head at join is only the fallback of the records commit the leader's ledger landing holds (D6).
The pair path is what the leader opens the member's worktree location from, to build that member's own seams for D6's checks:
its reconcile and owner reads, its beam land check, its pull-request record path and lock, its beam landing record, its beam mode, its origin and whether its task branch has commits beyond the parent, as the run's own Finalize builds them from its worktree.
An entry is live while its owner process is, judged by pid and start time as `parentlock` judges a holder.
A run's Finalize removes its own entry when it ends at Done; a halt or Stuck keeps it.
A run that joins again at Finalize rewrites its entry with its new task head.
It keeps a `failed` or `conflict` outcome until it lands alone (D8), whatever its fixer or merge commits since,
and it keeps a `landed` outcome only while its task branch tip is still the entry's task head.
A leader removes a dead owner's entry only while it has no outcome and only after D6's backstop ran; an entry with an outcome stays until its own run, resumed, reads it.
The directory also holds at most one move mark (D6).
Rationale: the queue needs no daemon, since the runs already block in the parent lock's wait;
an outcome outlives a driver restart so a resumed run still learns what happened to it, and a lost outcome costs no landing, since the parent is read first (D8).

### D3 — The parent lock's holder leads the batch

The run that takes the parent lock with no outcome on its entry, and whose work the parent does not hold (D8), is the leader.
Under the lock it syncs the parent as today and takes its batch: its own entry first, then live entries with no outcome in join order, up to `landing.batch_max` entries.
It skips an entry whose run a pause covers or whose task branch tip is not the entry's task head;
it judges each member again before the move (D6).
A batch of one runs today's merge and gate under the lock, with today's fixer.
Rationale: the lock already serializes landings on a parent and lives on its owner machine, so leadership needs no new election;
a batch of one keeps today's single-run behaviour, fixer included.

### D4 — Fabric builds the batch as a chain of landing commits with no worktree

A new fabric verb takes the parent tip and the batch's entries in order.
For each entry it merges the entry's task head into the previous chain commit with `git merge-tree --write-tree` (through `internal/gitrepo/merge.go`'s merge-tree call) and writes the landing commit with `git commit-tree` through a new `gitrepo` call.
The landing commit has the previous chain commit as its one parent when the entry squashes, and the entry's task head as its second when it true-merges; its message is the entry's landing message with its `Lym-Landing` trailer.
An entry whose merge is not clean is left out of the chain and gets outcome `conflict`; the next entry merges onto the chain commit before it.
An entry whose task head the chain commit before it already holds (`Fabric.HoldsWork`) is left out of the chain with no outcome, so no empty second landing commit is written.
Batch-gate failures a rerun did not repeat are listed in no landing commit, since the commits are built before the gate;
the leader logs them at Warn with the chain commit.
Rationale: each landing keeps today's shape and trailer, one commit per run, while each chain commit's tree is the tree its prefix lands as.

### D5 — The leader gates a chain commit in its own task worktree

Fabric checks the leader's task worktree out detached at the chain commit under test, the leader runs its Finalize gate key through `internal/verifytree` there, and fabric checks the worktree back out on its task branch on every way out, a failed gate included.
Before its first detach of a lock hold, the leader writes a scratch marker naming its task branch, and it removes the marker after its last restore, so the marker stands across every chain commit a bisection or a rebuild checks out.
After each checkout and after the restore, the leader removes the ignored files left in directories the checkout deleted since the commit it left, through the seam the merge-in uses (`Deps.RemoveLeftovers`), before the clean check.
While the worktree is detached, its readers act as follows:

- every task-head read on the batch path, and every read after the restore, resolves the task branch ref, never the worktree's HEAD;
- `ParentHoldsTaskWork` and the owner checks resolve the task branch ref;
- the pair's pusher (`readLocalBranch` and the push's `CurrentBranch`) and its refusal read (`RefusedPushStands`) hold off the code side while the marker stands;
- the gate's darn scope read (`darnTreeScope`) keeps reading HEAD.

The run's resume, which every row's resume and `lym loom goto` pass through, restores a task worktree found clean and detached while the marker stands, at whatever commit, onto its task branch before any row runs, and removes the marker.
A dirty detached worktree is Stuck naming the dirty paths, with the way forward to clean them and run `lym loom resume`.
A detached worktree with no marker is Stuck naming its commit, with the way forward to check the task branch out in the worktree and run `lym loom resume`.
The task branch ref is never moved by the batch.
Bound: the gate config is the leader's own, as its worktree's layers resolve it, so a run-level gate override on the leader applies to the whole batch;
a tree-scoped leader's batch gate scopes to the trees every member of the prefix touches, since the darn scope read diffs the pair's merge base against the chain commit at HEAD.
Rationale: the task worktree is a resolvable lym worktree with the hub's gate slots, so the gate needs no new worktree kind;
the gate key's `scope: all` and its inert-path check against the parent cover every run in the chain.

### D6 — A passing prefix lands with one guarded fast-forward and its ledger landings

Landing a prefix of the chain, the leader runs today's owner checks for its own run and the lock re-read.
For each other member of the prefix, with that member's own seams (D2), it then judges the checks `landUnderLock` runs for its own run:
`ownerChecks`' reconcile verdict and owner read, `landCheck`, `handedOverStop`'s owner read, and landingshed's pull-request resolver.
It judges them as reads alone and acts on no member's behalf:
a hand-over, a follow-up removal, a task-branch fast-forward, and a pull request's opening, closing or record removal stay with the member's own Finalize,
and a member whose check cannot be judged without such an action is left out.
Right before the move it re-reads each member's pause cover, liveness and task branch tip.
A member that fails any of these, or that the resolver sends through a pull request, is left out with no outcome:
the leader builds the chain again without it and gates again, and the member's own Finalize meets the same check under the lock and stops, or lands alone, as today.
A parent whose head moved is no landing: the leader builds the chain again on the new head and gates again.
One lock hold makes at most `landing.batch_max` such rebuilds, for a moved head or a left-out member;
past that the leader lands alone through today's merge and gate.
Before the move, the leader writes a move mark in the queue directory: the chain base, each prefix commit, and each member's slug and records commit.
A member's records commit is the oldest one origin lacks, as `oldestHeldRecords` picks it today, falling back to the entry's records head at join.
Fabric then moves the parent pair to the prefix's last commit only if the parent's head is still the chain's base.
Right after the move, still under the lock, the leader writes one push-ledger landing per entry of the prefix, each with its own landing commit as the tip and the records commit the mark names, and sets each entry's outcome to `landed`.
It removes the mark once every ledger landing and outcome is written.
The leader then does its own run's remaining landing records and frees the lock.
A batched run's remaining landing records take its own landing commit, never the parent's head:
`Deps.RecordLanding`, the dobby's landed-commit record, gets that commit, the landed size and the empty-landing check measure from that commit's first parent, and `Deps.LandLocally` gets that commit too.
`Deps.LandLocally` on a run whose ledger landing stands rewrites it with the same tip,
and keeps its records commit while origin lacks it and the run's new records commit descends from it, as `oldestHeldRecords` does today,
so the ledger landing written at the move keeps its tip, and the call still commits the run's status records and marks its pair's push.
Backstop: every taker of the parent lock, a run or a fabric verb such as `lym fabric pull` or `lym fabric anchor`, reads a standing mark first, before a run prunes any dead entry (D2).
For each mark commit the parent holds and its upstream lacks, it writes a missing ledger landing with the mark's records commit and sets that entry `landed` with its landing commit, then removes the mark;
a mark whose commits the parent does not hold, left by a leader that died before the move, is removed with nothing written.
Bound: a member's records hold starts at the leader's ledger write, which holds the records commit the mark names;
a push of that member's records before the write is harmless, since no records commit before the ledger landing marks the run landed.
Rationale: the parent is never ahead of its upstream by a commit that is not an owed landing tip, which `syncParent`, the prime's pull and the parent's pusher require;
the guarded move lands exactly the gated tree or nothing.

### D7 — A failed batch is bisected under the same lock

A gate failure on a batch of n entries, after landingshed's rerun of a test-run failure, which the batch gate keeps, starts a binary search over the chain's prefixes with no new chain built:
the empty prefix counts as passing and the whole batch as failing, and each step gates the middle prefix, until a passing prefix of k entries is followed by a failing one of k+1.
Entry k+1 is the culprit.
The passing prefix of k entries lands as D6 says, when k is at least 1.
The culprit gets outcome `failed`, whether it is another run's entry or the leader's own; the entries after it stay with no outcome for the next leader.
A leader that is its own batch's culprit lands nothing, and the other entries stay with no outcome.
It merges the chain base into its task worktree under the lock, a merge chain commit 1 shows clean, so the worktree holds the tree the prefix-1 gate failed on and the merge record names that base.
It then writes its landing failure record from the prefix-1 gate's log and goes to its fixer on that log, so no gate runs twice on one tree.
The batch gate runs `verifyGate`'s check with no failure recorder and with a flaky callback of its own, which logs each failure the rerun did not repeat at Warn with the chain commit (D4) and adds nothing to the leader's flaky list.
So the batch gate writes no landing failure record or flaky record for the leader;
only a run's own gate on its own path writes them, and a leader that is its own culprit, from the prefix-1 log.
A runner crash or a memory-pressure stop halts the leader as today and sets no outcome.
Bound: a failure that comes only from two runs together blames the later one, whose own gate then runs on the parent holding the earlier;
a failure from ungated upstream commits `syncParent` brought onto the parent, or a tier timeout or build failure, which the rerun does not run again, blames entry 1 or the prefix where it first shows.
The blamed run's own gate then runs on its own path, as for any culprit, so nothing lands ungated.
Rationale: about log2(n) extra gates find the culprit and land every run before it in the same lock hold;
a run's own failure is fixed only by its own fixer, in its own worktree.

### D8 — A queued run acts on its outcome when it takes the lock

Before it acts on its outcome, a run that takes the lock runs D6's backstop and reads `ParentHoldsTaskWork` for its task branch tip.
A run whose work the parent holds, whatever its entry says, takes the held path, which never merges the parent into the task worktree and runs no gate:
it judges the owner checks as a held run does today, by the reconcile's ended and stopped verdicts alone, then `handedOverStop`;
it does its remaining landing records with its own landing commit, as D6 says, whose `Deps.LandLocally` call writes its ledger landing when none stands;
it removes its entry and ends Finalize.
Its own landing commit is its entry's `landed` commit while the parent's head holds that commit;
otherwise it is the newest commit on the parent's first-parent history, origin's included, whose `Lym-Landing` trailer names its slug.
With neither, the run records the parent's head with no landed size and its landing read as not empty, as a held run with no merged parent head records it today.
Otherwise:

- `landed` with its landing commit on the parent and its task branch tip equal to the entry's task head: the run takes the held path.
- `landed` with a landing commit the parent's head no longer holds, such as after an operator's reset of the parent, or with a task branch tip past the entry's task head: the run clears the outcome and acts as with none.
- `failed`: the run keeps the outcome, so no leader takes it, and runs today's merge and gate under the lock alone, fixer included, and lands alone; it leads no batch.
- `conflict`: the run frees the lock and merges the parent's tip into its task worktree with no lock held, through the resolver, as D1's merge does.
  It keeps its outcome, then takes the lock again at the front of the queue and runs today's merge, gate and fixer alone, and lands alone; it leads no batch.
  A merge under the lock that would need a conflict session, after a parent that moved meanwhile, frees the lock and runs the lock-free merge again first.
- No outcome: the run leads a batch (D3).

Rationale: every run ends its own Finalize, so its records and teardown stay its own, and a run on its own path is never batched with an unfixed head;
the parent, not the queue, says whether a run's work landed.

### D9 — `landing.batch_max` is added and `landing.finalize_lost_races` removed

`landing.batch_max` is the most entries one batch takes, default 4; 1 makes every run land alone under the lock.
A value below 1 reads as 1.
`landing.finalize_lost_races` leaves the template and `Config`, with no reinterpretation: the key's meaning does not change, it goes.
It joins landing's `RetiredKeys` as `CategoryCrossPair`, so the local and machine layers refuse it, and a default or worktree layer still setting it gets configengine's unknown-key warning.
Rationale: per chain built, four bounds a failed batch's bisection at two extra gates, while still taking a typical burst of ready runs in one gate.

### D10 — A pull-request landing never lands in a batch

A run known before the lock to need GitHub (D1) writes no entry and lands alone through today's path under the lock.
A pull-request landing is decided under the lock for every run:
a leader whose own decision sends it through a pull request lands alone and leads no batch, and the leader runs the resolver for each member before the move (D6).
A member the resolver sends through a pull request is left out of the chain and lands alone through its own Finalize.
Bound: a GitHub rule the resolver's read misses ends in a direct push GitHub refuses, as today.
Rationale: a pull request merges on GitHub one at a time, so a local chain could not land it as gated.

### D11 — The glossary gains `landing queue`

`internal/heddleengine/glossary.md` gains:
`- **landing queue**: the runs at Finalize waiting to land on one parent branch, which the parent lock's holder gates and lands together as one batch. Never: merge queue, merge train.`
Rationale: the term is new, and "batch" alone already names Webster's unit of cards.

### D12 — `PATTERN-landing` names the batch

`PATTERN-landing`'s rule says a Finalize lands its own run or a batch of its parent's landing queue, and its background file gains a section on the queue: who leads, how a batch is built, gated and bisected, and who writes each ledger landing.
`internal/landingshed/doc.go` replaces its lock-free-pass and lost-race paragraphs with the queue's flow.
Rationale: the leader lands other runs' work on the parent, which the rule must name.

## Constraints

- `PATTERN-landing`: D12 changes it.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the chain build, the detached checkout and its restore and the guarded parent move are fabric verbs, and `merge-tree` and `commit-tree` run through `gitrepo`.
  The task `fabric-core-merge-landing` later moves fabric's merge rules into `internal/fabriccore`; the new verbs sit beside today's merge code so that task moves them with it.
- `PATTERN-lym-files`: the queue's directory is never tracked, sits under the hub's `.lym` and is named by `lymdirs`; its lock path comes from `lock`.
  The queue is a cache: an absent entry or outcome reads as none, and a landed run is still found landed through the parent (D8).
- `PATTERN-tolerant-records`: queue entries are rewritten through `internal/state` and refuse no unknown key.
- `PATTERN-daemon-wakeups`: a queued run waits only in `parentlock`'s existing wait, which backs off from one second and wakes on the lock record's change.
- `PATTERN-owner-machine`: every run that lands on a parent runs on the parent's owner machine, so one hub-local queue per parent holds every run that can land there.
  A member's ownership is judged under the lock, with its own seams, before the move that lands it (D6).
- `PATTERN-config-layers`: `landing.batch_max` is a template key of `landingshed`'s config; the removed key needs no reinterpretation because no meaning changes.
- The push ledger's landing (`pushledger.Landing`) is today written per run through its `LandLocally` seam; D6 has the leader write it for each batch entry, its own run included, with that run's own landing commit as the tip, so that write takes the slug, tip and records commit as arguments rather than from the calling run;
a batched run's later `LandLocally` call takes its own landing commit too (D6).
  `fabricengine.Fabric.HoldsWork` judges a held landing by merge-tree tree equality, so it reads a squash chain commit as holding its run's work.
- The hub's Finalize gate key is tiers 0 to 2 at `scope: all` (the hub's `gate.yaml`, read 2026-11-28), so one gate on a chain commit covers every run in its prefix.
- The parent lock is released early only for an idle holder; a run at Finalize with a live verify keeps it (`landing.parent_lock_idle_release_min`, 10 minutes), so a leader's gates and the short gaps between bisection steps keep the lock.

## Auto-mode assumptions

- D1: the queue replaces the lock-free gate rather than running beside it.
- D9: `landing.batch_max` defaults to 4.
- D5: the batch is gated in the leader's own task worktree.

## Open risks

None.

## Acceptance criteria

- D1, D3: a single run at Finalize with no other queued run merges the parent, gates under the lock, and lands as today; no gate runs before it holds the lock. Tier 1, `internal/landingshed`.
- D2: an entry is written, read, rewritten with an unknown key kept, ignored once its owner process is gone, and kept past its owner's death while it carries an outcome. Tier 0, `internal/landingqueue` (new).
- D2: a run that ends Finalize at Done leaves no queue entry. Tier 1, `internal/landingshed`.
- D3: a leader skips an entry whose run is paused, whose task branch moved past its recorded head, or which carries an outcome, and takes at most `landing.batch_max` entries in join order after its own. Tier 1, `internal/landingshed`.
- D4: three task branches on one parent build a chain of three landing commits, each with its run's message and `Lym-Landing` trailer, one parent each when squashing and two when true-merging; a branch that conflicts with the chain is left out and the next one merges onto the commit before it. Tier 1, `internal/fabricengine`.
- D5: the leader's gate runs with its task worktree at the chain commit and leaves it on its task branch after a pass and after a failure; a run resumed at any row with its worktree clean and detached while the batch marker stands is put back on its task branch first. Tier 1, `internal/landingshed`.
- D5: a Finalize resumed with its task worktree dirty and detached is Stuck naming the dirty paths, and moves nothing. Tier 1, `internal/landingshed`.
- D5: a batch whose member deletes a directory holding ignored build output gates the chain commit and restores the leader clean. Tier 1, `internal/landingshed`.
- D5: a darn leader's batch gate over a chain commit runs targets for every member's trees. Tier 1, `internal/landingshed`.
- D6: three queued runs land with one gate run; the parent's tip is the commit the gate ran on; the push ledger holds one landing per run with its own landing commit before the lock is freed; a parent whose head moved after the chain was built is not moved. Tier 1, `internal/landingshed`.
- D6: a member whose entry reads as owned elsewhere, or ended on origin, after it joined is not moved onto the parent; the other members land. Tier 1, `internal/landingshed`.
- D6: a member paused, or whose task branch moved, after the batch was taken is not moved onto the parent. Tier 1, `internal/landingshed`.
- D6: after a leader dies between the move and the ledger write, and its dead entries would be pruned, the next run to take the lock writes one ledger landing per moved commit with that member's records commit. Tier 1, `internal/landingshed`.
- D6: after a batch of three lands, each run's `Deps.LandLocally` and `Deps.RecordLanding` calls, the leader's included, take that run's own landing commit, never the parent's head. Tier 1, `internal/landingshed`.
- D6: a parent ahead of its upstream by a chain of three landing commits, whose ledger holds one landing per run with that run's own landing commit as its tip, reads as ahead by owed tips alone, and a run recorded again through `landLocally` with its own landing commit keeps that tip. Tier 1, `internal/loomcli`.
- D6: a member's pair pusher sends nothing past the records commit the leader's ledger landing holds until the parent's push is made. Tier 1, `internal/landingshed`.
- D6, D8: a run woken with outcome `landed` runs no merge and no gate and ends Finalize; a run that takes the lock after a leader died between the move and the ledger write writes the missing landings; a run whose `landed` commit the parent's head no longer holds, as after an operator's reset of the parent, leads a batch. Tier 1, `internal/landingshed`.
- D7: a batch of four whose third run fails lands the first two, gives the third outcome `failed`, and leaves the fourth with no outcome; the batch failure writes no landing failure record for the leader; a run with outcome `failed` gates alone and runs its own fixer. Tier 1, `internal/landingshed`.
- D7: a batch whose culprit is the leader lands nothing, gives the leader outcome `failed` and leaves the other entries with no outcome. Tier 1, `internal/landingshed`.
- D7: while a leader that was its batch's culprit runs its fixer with the lock freed, the next leader's batch does not take its entry. Tier 1, `internal/landingshed`.
- D8: a run with outcome `conflict` merges the parent itself, lands alone, and is taken into no batch meanwhile; its conflict session runs with no parent lock held. Tier 1, `internal/landingshed`.
- D8: a run whose work the parent holds, whatever its entry says, runs no merge and no gate, writes no landing commit and ends Finalize. Tier 1, `internal/landingshed`.
- D8: a run whose task branch gained commits after it was marked `landed` lands them. Tier 1, `internal/landingshed`.
- D9: `landing.batch_max` reads 4 by default and 1 for a value below 1; a hub default layer setting `landing.finalize_lost_races` loads with the unknown-key warning. Tier 0, `internal/landingshed`.
- D10: a run known before the lock to need GitHub writes no queue entry, and a leader never lands a run whose landing goes through a pull request in its batch. Tier 1, `internal/landingshed`.
- D10: a member whose pull-request record appears after it joined is left out of the batch and lands alone through its pull request. Tier 1, `internal/landingshed`.
- Review: D11 — `internal/heddleengine/glossary.md` carries the `landing queue` line as D11 gives it.
- Review: D12 — `PATTERN.md`'s `PATTERN-landing` line, `pattern/PATTERN-landing.md` and `internal/landingshed/doc.go` describe the queue, its leader, the batch's gate and bisection and the ledger landings, and drop the lock-free pass and lost races.

## PATTERN changes

- `PATTERN-landing` — changes: Landing on a local parent branch or deciding on a pull request: only in a run's Finalize under `internal/parentlock`'s lock, alone or in a queue batch; landingshed's one resolver decides it; a pull request's review ends only on the operator's word, whatever the run's autonomy.

## Notes for the plan writer

- `landingshed.Finalize.Call` holds the lock-free pass and the lost-race loop D1 removes; `internal/landingshed/parentguard.go` holds the lock wait the queued run keeps.
- `landingshed.Deps.RecordLanding` writes the dobby's landed-commit record, not the push ledger; the ledger landing goes through `landingshed.Deps.LandLocally`, wired in `internal/loomcli/landingdeps.go`.
