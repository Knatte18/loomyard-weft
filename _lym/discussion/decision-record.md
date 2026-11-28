## Goal

A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips

Takes: D6's ledger steps, as the Constraints' push-ledger bullet words them: the push-ledger landing write in internal/loomcli/landingdeps.go (landLocally and recordHeldLanding, calling runpusher.RecordLanding with a pushledger.Landing) takes the run's slug, the landing tip and the records commit as arguments instead of reading them from the calling run's worktree, the records commit picked as oldestHeldRecords picks it today; landingshed's Deps.LandLocally takes the run's own landing commit as its tip, which a lone run's Finalize (internal/landingshed/finalize.go, its LandLocally call) passes, and which for a lone run is the commit it lands today; on a run whose ledger landing stands, LandLocally rewrites it with the same tip, keeps its records commit while origin lacks it and the run's new records commit descends from it, and still commits the run's status records and marks its pair's push. No queue, no batch and no leader: a run's Finalize still writes only its own landing.
Delivers: One push-ledger landing writer that takes another run's slug, tip and records commit, with today's single-run landing unchanged; a parent ahead of origin by a chain of runs' landing commits, each the tip of its own run's ledger landing, reads as ahead by owed tips alone.
Exports: Fixed for landing-queue-batch: the loomcli writer of one push-ledger landing taking the parent branch, the run's slug, its landing commit and its records commit, and the landingshed Deps seam a Finalize reaches it through; the plan names both and the change description lists their signatures.

## Scope

The part changes these packages:

- internal/loomcli (about 220 changed lines)
- internal/landingshed (about 30 changed lines)

## Decisions

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

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - wired-landlocally-failed-head-read-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D6: a parent ahead of its upstream by a chain of three landing commits, whose ledger holds one landing per run with that run's own landing commit as its tip, reads as ahead by owed tips alone, and a run recorded again through `landLocally` with its own landing commit keeps that tip. Tier 1, `internal/loomcli`.
