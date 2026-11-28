---
format: 5
approved: false
language: go
---

# Plan: Finalize joins a landing queue, and the parent lock's holder lands a batch with one gate run

A run at Finalize merges the parent with no lock and no gate, writes its entry in the parent's landing queue and waits in the parent lock's queue.
The run that takes the lock with no outcome leads: it builds a chain of landing commits, gates the chain commit once in its own detached task worktree, bisects a failure under the same lock, and lands a passing prefix with one guarded fast-forward and one push-ledger landing per run.
A run woken with an outcome acts on it; a batch of one, a pull-request landing and a run with outcome `failed` or `conflict` land alone as today.
The cards build the queue store, the git and fabric verbs, then Finalize's queue join, leader and bisection, the resume restore, the fabric verbs' backstop and the docs.

## Card Index

1 — landing-queue-store — add internal/landingqueue: entries, outcomes, the move mark and the backstop
2 — gitrepo-commit-tree — add the git commit-tree call beside MergeTree
3 — fabric-landing-chain — add fabric's chain build, clean-merge read, guarded move and newest-landing read
4 — fabric-detached-checkout — add the detached checkout, its marker and restore, and branch-ref reads while detached
5 — finalize-lock-free-merge — remove the lock-free gate and the lost-race loop, keep the lock-free merge
6 — landing-batch-max-key — add landing.batch_max and retire landing.finalize_lost_races
7 — finalize-queue-entry — join the queue, run the backstop and act on the entry's outcome under the lock
8 — finalize-batch-leader — lead a batch: take it, build and gate the chain, judge members and land the prefix
9 — finalize-batch-bisection — bisect a failed batch gate under the same lock
10 — resume-restores-detached — restore a detached task worktree on resume and stop on one Finalize cannot restore
11 — fabric-lock-backstop — run the backstop where fabric's verbs take the parent lock
12 — landing-queue-docs — the glossary line, PATTERN-landing and landingshed's package doc

## Shared Decisions

### Decision: fabric-lock-takers

- **Decision:** D6's backstop runs in `internal/fabriccli`'s `acquireBranchLock`, which both `lym fabric anchor` and `lym fabric pull --hub` take the parent lock through.
- **Rationale:** the decision record's Goal says `lym fabric pull` takes no parent lock, but `runHubPull` takes it through `acquireBranchLock` with the verb `pull`; D6 names every taker of the lock, so one call there covers both verbs.
- **Applies to:** card 11

### Decision: queue-off-in-struct-literal-tests

- **Decision:** every new `landingshed.Deps` seam is optional; with no `Queue` a Finalize writes no entry and lands alone, and with no `MemberDeps` it takes no batch.
- **Rationale:** the package's struct-literal tests keep today's single-run behaviour, while `internal/loomcli`'s `landingDeps` fills every seam and its reflection drift test holds it filled.
- **Applies to:** cards 7, 8, 10

### Decision: detach-marker

- **Decision:** the batch marker of D5 is fabric's detach marker, a file under the task worktree's `.lym` that `lymdirs.DetachMarkerName` names and fabric reads and writes.
- **Rationale:** fabric's pusher reads must see it, and fabric learns nothing of the landing queue, so the marker names only the task branch.
- **Applies to:** cards 4, 8, 10

## PATTERN changes

- `PATTERN-landing` — changes: Landing on a local parent branch or deciding on a pull request: only in a run's Finalize under `internal/parentlock`'s lock, alone or in a queue batch; landingshed's one resolver decides it; a pull request's review ends only on the operator's word, whatever the run's autonomy.
