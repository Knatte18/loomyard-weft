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
It resolves whether it lands through a pull request (D10); a run that does not commits its records, writes its queue entry (D2) and waits for the parent lock through `parentlock.Store`'s ticket queue, as today.
Under the lock it acts on its entry's outcome (D8).
The lock-free gate, the lost-race comparison and the lost-race count are removed.
Rationale: every landing on a busy parent made each waiting run's lock-free gate stale, and those gates competed with the lock holder for gate slots;
the merge stays lock-free so a conflict session never holds the lock for a run that merged cleanly.

### D2 — The queue is one entry file per run under the hub's `.lym`

`internal/landingqueue` (new) owns a directory per parent branch, named by `lymdirs` and keyed by the branch's hex like `internal/parentlock`'s records, holding one JSON entry per queued run, written through `internal/state` under a `lock` file lock.
An entry carries the run's slug, pair path, run id, pid and start time, its task head, its records commit at join, the landing message from its change description, whether its landing squashes or true-merges, its join time, and an outcome: none, `landed` with its landing commit, `conflict`, or `failed` with the chain commit it failed on.
An entry is live while its owner process is, judged by pid and start time as `parentlock` judges a holder.
A run removes its own entry when its Finalize ends.
A leader removes a dead owner's entry only while it has no outcome; an entry with an outcome stays until its own run, resumed, reads it.
Rationale: the queue needs no daemon, since the runs already block in the parent lock's wait;
an outcome outlives a driver restart so a resumed run still learns what happened to it.

### D3 — The parent lock's holder leads the batch

The run that takes the parent lock with no outcome on its entry is the leader.
Under the lock it syncs the parent as today and takes its batch: its own entry first, then live entries with no outcome in join order, up to `landing.batch_max` entries.
It skips an entry whose run a pause covers or whose task branch tip is not the entry's task head.
A batch of one runs today's merge and gate under the lock, with today's fixer.
Rationale: the lock already serializes landings on a parent and lives on its owner machine, so leadership needs no new election;
a batch of one keeps today's single-run behaviour, fixer included.

### D4 — Fabric builds the batch as a chain of landing commits with no worktree

A new fabric verb takes the parent tip and the batch's entries in order.
For each entry it merges the entry's task head into the previous chain commit with `git merge-tree --write-tree` (through `internal/gitrepo/merge.go`'s merge-tree call) and writes the landing commit with `git commit-tree` through a new `gitrepo` call.
The landing commit has the previous chain commit as its one parent when the entry squashes, and the entry's task head as its second when it true-merges; its message is the entry's landing message with its `Lym-Landing` trailer.
An entry whose merge is not clean is left out of the chain and gets outcome `conflict`; the next entry merges onto the chain commit before it.
Rationale: each landing keeps today's shape and trailer, one commit per run, while each chain commit's tree is the tree its prefix lands as.

### D5 — The leader gates a chain commit in its own task worktree

Fabric checks the leader's task worktree out detached at the chain commit under test, the leader runs its Finalize gate key through `internal/verifytree` there, and fabric checks the worktree back out on its task branch on every way out, a failed gate included.
Every task-head read on the batch path, and every read after the restore, resolves the task branch ref, never the worktree's HEAD.
Finalize's entry and resume restore a task worktree found clean and detached at a commit other than its task branch tip before any other step; a dirty one is Stuck naming the paths.
The task branch ref is never moved by the batch.
Bound: the gate config is the leader's own, as its worktree's layers resolve it, so a run-level gate override on the leader applies to the whole batch.
Rationale: the task worktree is a resolvable lym worktree with the hub's gate slots, so the gate needs no new worktree kind;
the gate key's `scope: all` and its inert-path check against the parent cover every run in the chain.

### D6 — A passing prefix lands with one guarded fast-forward and its ledger landings

Landing a prefix of the chain, the leader runs today's owner checks and lock re-read, then has fabric move the parent pair to the prefix's last commit only if the parent's head is still the chain's base.
A parent whose head moved is no landing: the leader builds the chain again on the new head and gates again.
Right after the move, still under the lock, the leader writes one push-ledger landing per entry of the prefix, each with its own landing commit as the tip and its entry's records commit, and sets each entry's outcome to `landed`.
Each run's landed size and empty-landing check measure from its landing commit's first parent.
The leader then does its own run's remaining landing records and frees the lock.
Any run that takes the parent lock first writes a missing ledger landing for each chain commit past the parent's upstream whose entry is `landed` or whose `Lym-Landing` trailer names a queued entry, so a leader that died after the move loses no landing.
Bound: a member's records commits from its join on are held back until its landing is pushed, because its join-time records commit is the one the ledger holds.
Rationale: the parent is never ahead of its upstream by a commit that is not an owed landing tip, which `syncParent`, the prime's pull and the parent's pusher require;
the guarded move lands exactly the gated tree or nothing.

### D7 — A failed batch is bisected under the same lock

A gate failure on a batch of n entries, after verifytree's one rerun, starts a binary search over the chain's prefixes with no new chain built:
the empty prefix counts as passing and the whole batch as failing, and each step gates the middle prefix, until a passing prefix of k entries is followed by a failing one of k+1.
Entry k+1 is the culprit.
The passing prefix of k entries lands as D6 says, when k is at least 1.
A culprit that is another run's entry gets outcome `failed`; the entries after it stay with no outcome for the next leader.
A culprit that is the leader's own entry lands nothing, and the leader runs today's merge, gate and fixer alone; the other entries stay with no outcome.
The batch gate writes no landing failure record or flaky record for the leader; only a run's own gate on its own path does.
A runner crash or a memory-pressure stop halts the leader as today and sets no outcome.
Bound: a failure that comes only from two runs together blames the later one, whose own gate then runs on the parent holding the earlier.
Rationale: about log2(n) extra gates find the culprit and land every run before it in the same lock hold;
a run's own failure is fixed only by its own fixer, in its own worktree.

### D8 — A queued run acts on its outcome when it takes the lock

- `landed` with its landing commit on the parent: the run merges and gates nothing, writes its ledger landing if it is missing, does its remaining landing records, removes its entry and ends Finalize.
- `landed` with its landing commit not on the parent, left by a leader that died before the move: the run clears the outcome and acts as with none.
- `failed` or `conflict`: the run keeps the outcome, so no leader takes it, and runs today's merge and gate under the lock alone, resolver and fixer included, and lands alone; it leads no batch.
- No outcome: the run leads a batch (D3).

Rationale: every run ends its own Finalize, so its records and teardown stay its own, and a run on its own path is never batched with an unfixed head.

### D9 — `landing.batch_max` is added and `landing.finalize_lost_races` removed

`landing.batch_max` is the most entries one batch takes, default 4; 1 makes every run land alone under the lock.
A value below 1 reads as 1.
`landing.finalize_lost_races` leaves the template and `Config`, with no reinterpretation: the key's meaning does not change, it goes, and a hub layer that still sets it is warned of as a retired key.
Rationale: four bounds a failed batch's bisection at two extra gates, while still taking a typical burst of ready runs in one gate.

### D10 — A pull-request landing never joins a queue

A run whose landing the resolver sends through a pull request, or which is known to need GitHub, lands alone through today's path under the lock.
A leader never takes such a run.
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
- `PATTERN-tolerant-records`: queue entries are rewritten through `internal/state` and refuse no unknown key.
- `PATTERN-daemon-wakeups`: a queued run waits only in `parentlock`'s existing wait, which backs off from one second and wakes on the lock record's change.
- `PATTERN-owner-machine`: every run that lands on a parent runs on the parent's owner machine, so one hub-local queue per parent holds every run that can land there.
- `PATTERN-config-layers`: `landing.batch_max` is a template key of `landingshed`'s config; the removed key needs no reinterpretation because no meaning changes.
- The push ledger's landing (`pushledger.Landing`) is today written per run through its `LandLocally` seam; D6 has the leader write it for each batch entry, so that write takes the slug, tip and records commit as arguments rather than from the calling run.
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
- D3: a leader skips an entry whose run is paused, whose task branch moved past its recorded head, or which carries an outcome, and takes at most `landing.batch_max` entries in join order after its own. Tier 1, `internal/landingshed`.
- D4: three task branches on one parent build a chain of three landing commits, each with its run's message and `Lym-Landing` trailer, one parent each when squashing and two when true-merging; a branch that conflicts with the chain is left out and the next one merges onto the commit before it. Tier 1, `internal/fabricengine`.
- D5: the leader's gate runs with its task worktree at the chain commit and leaves it on its task branch after a pass and after a failure; a Finalize resumed with its worktree clean and detached is put back on its task branch first. Tier 1, `internal/landingshed`.
- D6: three queued runs land with one gate run; the parent's tip is the commit the gate ran on; the push ledger holds one landing per run with its own landing commit before the lock is freed; a parent whose head moved after the chain was built is not moved. Tier 1, `internal/landingshed`.
- D6, D8: a run woken with outcome `landed` runs no merge and no gate and ends Finalize; a run that takes the lock after a leader died between the move and the ledger write writes the missing landings; a run whose `landed` commit is not on the parent leads a batch. Tier 1, `internal/landingshed`.
- D7: a batch of four whose third run fails lands the first two, gives the third outcome `failed`, and leaves the fourth with no outcome; the batch failure writes no landing failure record for the leader; a run with outcome `failed` gates alone and runs its own fixer. Tier 1, `internal/landingshed`.
- D8: a run with outcome `conflict` merges the parent itself, lands alone, and is taken into no batch meanwhile. Tier 1, `internal/landingshed`.
- D9: `landing.batch_max` reads 4 by default and 1 for a value below 1; a hub layer setting `landing.finalize_lost_races` loads with the retired-key warning. Tier 0, `internal/landingshed`.
- D10: a run whose landing goes through a pull request writes no queue entry, and a leader never takes it. Tier 1, `internal/landingshed`.
- Review: D11 — `internal/heddleengine/glossary.md` carries the `landing queue` line as D11 gives it.
- Review: D12 — `PATTERN.md`'s `PATTERN-landing` line, `pattern/PATTERN-landing.md` and `internal/landingshed/doc.go` describe the queue, its leader, the batch's gate and bisection and the ledger landings, and drop the lock-free pass and lost races.

## PATTERN changes

- `PATTERN-landing` — changes: Landing on a local parent branch or deciding on a pull request: only in a run's Finalize under `internal/parentlock`'s lock, alone or in a queue batch; landingshed's one resolver decides it; a pull request's review ends only on the operator's word, whatever the run's autonomy.

## Notes for the plan writer

- `landingshed.Finalize.Call` holds the lock-free pass and the lost-race loop D1 removes; `internal/landingshed/parentguard.go` holds the lock wait the queued run keeps.
- `landingshed.Deps.RecordLanding` writes the dobby's landed-commit record, not the push ledger; the ledger landing goes through `landingshed.Deps.LandLocally`, wired in `internal/loomcli/landingdeps.go`.
