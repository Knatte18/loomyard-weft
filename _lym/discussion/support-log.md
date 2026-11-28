## Interview

Autonomous session: no operator answered; every question below was settled by an auto-pick, recorded in the question ledger.

### Turn 1 — the board and the code

Read the entry `landing-queue`, the notes `landing-lock-effect-measure` and `lock-holder-gates-first`, the task `fabric-core-merge-landing`, the project `test-speed` and `lym loom peers`.
No `Decision for landing-queue:` note exists.
The entry releases the queue ahead of the measurement in `landing-lock-effect-measure` (operator, 2026-11-28), so that note's "decides whether the queue is still worth building" is settled.

Six exploration subagents covered Finalize's flow, the parent lock, pull-request landings, the gate and its caches, the run lifecycle and the dobby, and fabric's merge mechanics.
Findings the design rests on:

- Finalize merges the parent tip into the task worktree, gates lock-free, compares the parent head under the lock, and after `landing.finalize_lost_races` merges and gates under the lock; the landing is a squash merge commit by default (`landing.squash: true`) with a `Lym-Landing` trailer.
- The parent lock is a per-branch file record with a FIFO ticket queue, a front lane, pause yielding and fswatch wakeups, hub-local on the owner machine.
- Each run records its own landing through `Deps.RecordLanding`, carrying its own records commit; the parent's pusher pushes and marks the board entry done per landing.
- `gitrepo.Repo.MergeTree` wraps `git merge-tree --write-tree`; nothing wraps `commit-tree` for merges.
- `lymcwd` requires a worktree to be a direct child of the hub, so a gate in a temporary worktree is no resolvable lym worktree.
- The hub's Finalize gate key is tiers 0 to 2, `scope: all`; `beam.pull_request` is `off`; `PATTERN-landing`'s rule text cap is 280 characters (`internal/pattern/check.go`).

### Turn 2 — design, auto-picked

Questions and picks, each in the ledger: where the queue lives (in Finalize, led by the lock holder), how the batch is built (a chain of landing commits by merge-tree and commit-tree), where it is gated (the leader's own task worktree, detached), how a failure is split (halving prefixes), how a culprit reaches its fixer (an outcome on its entry, read on its own wake-up), what happens to the lost-race loop (removed), the batch cap (4), and pull-request landings (never batched).

### Turn 3 — lens review

Six forks, every one returned a report.

- interactions 1, walkthrough 1, bound-breaker 2, leanness 1 (BLOCKING/MEDIUM, merged): outcome `failed` unreachable because every halved prefix starts with the leader — fixed: D7 is now a binary search for the first failing prefix in the same lock hold, landing the passing prefix and marking the culprit; D7 criterion matches.
- interactions 2, existing-paths 1, walkthrough 2 (BLOCKING, merged): members' ledger landings written late leave the parent ahead by non-owed commits — fixed: D6 has the leader write every entry's push-ledger landing right after the move, under the lock, with each entry's join-time records commit (D2), plus a backstop for a leader dying in between.
- existing-paths 1 (part): `Deps.RecordLanding` is the dobby's scratch, not the ledger — fixed: D6 no longer names it; Constraints and Notes name `LandLocally`.
- bound-breaker 1 (BLOCKING): a `landed` wake-up runs `mergeAndGate` again — fixed: D8 `landed` merges and gates nothing.
- interactions 3, bound-breaker 4 (MEDIUM): dead-entry removal destroys the outcome recovery needs — fixed: D2 keeps an entry with an outcome until its run reads it.
- interactions 6, bound-breaker 3, walkthrough 6 (MEDIUM/LOW): stale `landed` outcome strands its run — fixed: D8 clears it and the run acts as with none.
- interactions 4, walkthrough 3 (MEDIUM): a `failed`/`conflict` run clearing its outcome can be re-batched with an unfixed head — fixed: D8 keeps the outcome until it lands alone and it leads no batch.
- existing-paths 2 (MEDIUM): task-head reads see the detached HEAD — fixed: D5 resolves the task branch ref on the batch path.
- existing-paths 3 (MEDIUM): the already-landed path writes no ledger landing — fixed: D8's `landed` path is a new path that writes a missing one.
- walkthrough 4 (MEDIUM): landed size and empty check measured from the wrong base — fixed: D6 measures from each landing commit's first parent.
- walkthrough 5 (MEDIUM): batch failure written to the leader's landing failure record — fixed: D7 says the batch gate writes none.
- bound-breaker 5 (MEDIUM): parent head moved between chain build and move — fixed: D6 moves the parent only from the chain's base, else builds and gates again.
- interactions 5, interactions 7 (LOW): which entries get `landed`, and what happens to the rest when the leader is the culprit — fixed in D6 and D7.
- bound-breaker 6 (LOW): a culprit can cost extra gates across leaders — fixed by the in-lock search; the claim is now per failed batch.
- existing-paths 4, 5 (LOW): held, no change.
- leanness 2 (MEDIUM): Open risks held D5/D7 bounds — fixed: moved into D5 and D7 as bounds; Open risks is now none.
- leanness 3, 4, 5, 6, 7, 8, 9 (LOW): repeated text in Auto-mode, Scope, Constraints, Notes and Goal — fixed: Auto-mode names ids, PATTERN constraint points to D12, the `beam.pull_request` motivation and the MergeTree note are cut, the Goal keeps the aim; the Scope D10 pointer stays as the one in/out line.
- leanness 10 (LOW): "or `internal/loomcli`" left the test home open — fixed: `internal/landingshed`.
- board-scope 1 (LOW): side-by-side prefixes declined without naming the entry's "may" line — fixed in Scope Out.

## Rejected alternatives

- **The dobby hosts the queue.** The parent beam's dobby is long-lived per parent, but its passes are short ticks, it would need a process of its own for a long gate, and landing outside a run's Finalize breaks `PATTERN-landing` outright. Leading from the lock holder needs no new process.
- **Keep the lock-free gate beside the queue.** Each landing makes every waiting run's lock-free gate stale, and those gates take the slots the leader needs; the entry names exactly that cost.
- **Gate the batch in a temporary worktree.** `lymcwd` resolves only a direct child of the hub, so the gate would run unslotted or with no hub config; a new worktree kind per parent is far more machinery than a detached checkout of the leader's worktree.
- **Gate the prefixes side by side.** With the hub's gate slots it multiplies slot contention, the fault the entry reports; halving costs about log2(n) extra gates only when a batch fails.
- **Land the batch through `Fabric.Merge` run by run.** Each merge would be computed again on the parent and could differ from the gated tree; fast-forwarding to the chain commits lands exactly what was gated.
- **The leader records every run's landing.** Each landing record carries its own run's records commit, which only that run knows; each run recording its own keeps the pusher and the board unchanged.
- **Mark the culprit and run its fixer from the leader.** A fixer works only in its run's own worktree, so the culprit's own Finalize runs it.
- **Halve to the first half and leave the rest for the next leader.** Every halved prefix starts with the leader, so a member could never be found as the culprit in its batch; a binary search for the first failing prefix finds it in the same lock hold.
- **Members write their own ledger landings on waking.** Until they do, the parent is ahead of upstream by commits that are no owed tip, which the next `syncParent`, the prime's pull and the pusher refuse.
- **Larger default batch (8 or the dobby's `max_runs`).** A failed batch of eight costs up to four extra gates under the lock; four takes a usual burst in one gate.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the queue live? — In Finalize: queue entries under the hub's `.lym`, the parent lock's holder leads. Rationale: no new process, landing stays in a run's Finalize under the lock.
- Q2 (auto-pick): Does the lock-free gate stay? — No; the merge-in stays lock-free, the gate moves to the leader. Rationale: the stale gates are the cost the entry names.
- Q3 (auto-pick): How is the batch tree built? — A chain of landing commits by merge-tree and commit-tree through fabric and gitrepo. Rationale: the parent's new tip is the commit gated.
- Q4 (auto-pick): Where does the batch gate run? — In the leader's own task worktree, detached at the chain tip, restored on every exit and on resume. Rationale: only a hub-child worktree resolves for the gate.
- Q5 (auto-pick): How is a failure split? — A binary search for the first failing prefix under the same lock; the prefix before it lands. Rationale: the entry's bisection, at about log2(n) gates, revised after the lens review.
- Q6 (auto-pick): How does a culprit reach its fixer? — An outcome `failed` on its entry; its own Finalize runs today's path alone. Rationale: a fixer works only in its own worktree.
- Q7 (auto-pick): What is the cap? — `landing.batch_max`, default 4, 1 disables batching. Rationale: bounds bisection at two extra gates.
- Q8 (auto-pick): What happens to `landing.finalize_lost_races`? — Removed, no reinterpretation; the hub base does not set it. Rationale: the loop it bounds is gone.
- Q9 (auto-pick): Do pull-request landings batch? — No. Rationale: GitHub merges them one at a time, so they cannot land as gated.
- Q10 (auto-pick): Who writes each ledger landing? — The leader, for every entry it lands, right after the move; each run does its other landing records itself. Rationale: the parent is never ahead by a non-owed commit; revised after the lens review.
- Q11 (resolved): Is the entry outdated by `landing-lock-only-for-land`? — No: the entry's 2026-11-28 section records the operator releasing it after that change.
