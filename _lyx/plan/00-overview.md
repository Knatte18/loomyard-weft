---
format: 5
approved: false
language: go
---

# Plan: batten-cancel-verb

Add `lyx batten cancel <slug> [--remove]`, which ends a batten run for good from the hub's prime whatever its state: it ends the run's processes and strands, clears the board claim, removes batten's records and scratch, and with `--remove` removes the pair, snapshots its uncommitted paths and archives and deletes its task branches.
Batten's pre-run refuses a run a cancel is ending, a dirty source and a start over a kept pair's left parts before it writes anything, and Worktree-Create halts with a precondition reason the Go driver hands back for a dirty source or a left task branch.
The cards build the leaf pieces first (shedrun paths, proc, fabric reads and removal, shed and driver seams, Worktree-Create's halt, pairteardown's cancel phase), then batten's marker, guards and pre-drive check, then the cancel itself, its verb and its docs.

## Card Index

1 — cancel-marker-paths — declare the cancel marker's path and lock path in shedrun
2 — proc-creation-time — read a process's creation time by pid on Windows
3 — source-dirty-typed-error — the pair add's typed dirty-source refusal and its exported probe
4 — pair-parts-and-branch-discard — fabric's local pair-parts read, origin task-branch read and archive-and-delete of the task branch
5 — records-write-lock — export the combined records write lock hold
6 — shed-locked-check-and-precondition — shed's precondition prefix and the check Run and Step call under the run lock
7 — driver-precondition-row — the driver hands a precondition halt back
8 — worktree-create-precondition-halt — Worktree-Create halts on a PreconditionError
9 — batten-create-precondition-mapping — batten maps a dirty source and a left task branch to the precondition halt
10 — board-runlock-names-cancel — the board's run-lock refusal names the cancel
11 — cleanup-removal-split — split batten cleanup's per-slug removal from its locks and judgement
12 — teardown-origin-snapshot-and-claim-mode — origin-mode snapshot, a claim-leaving removal and the held-gone wording
13 — teardown-cancel-phase — pairteardown's cancel phase
14 — batten-cancel-marker — the cancel marker's write, read and liveness lock
15 — batten-start-guards — the pre-run's marker, dirty-source and left-parts refusals
16 — batten-pre-drive-check — the marker and seed re-read in the seeding closure and under shed's run lock
17 — batten-cancel-core — the cancel's decisions and order over injected seams
18 — batten-cancel-verb — the verb, its production seams and the judge-only reconcile
19 — docs-cancel — overview and orch stencil name the verb

## Shared Decisions

### Decision: cancel-core-over-seams

`cancelRun` (card 17) holds every decision and step of D2 to D9 over a struct of function seams, so one untagged table test reaches every branch, refusal and acceptance path with fakes; card 18 wires the seams over the hub, and one tier-1 integration test pins the no-pair acceptance path end to end.

### Decision: ended-reading-from-how

D2 splits an ended entry by whether it reads done on the fetched Bolt tip.
The run-scope reconcile already carries that reading on `EntryOutcome.How`, `done` for an entry done on the fetched Bolt and `removed` for one gone from it (`internal/fabricengine/originreconcile.go:200`), which `resumeRefusal`'s `endedReason` reads today, so the cancel reads `How` and makes no second Bolt read.

### Decision: gone-worktree-strands

D4 lists the phase's step 3, the strand stop, for a gone task worktree too.
The strands are listed from the task worktree's reed directory, which goes with the worktree, so for a gone worktree step 3 finds none to stop through shuttle, and step 6's session end by name ends their panes.

### Decision: start-and-drive-guards-as-seams

The start guard (card 15) and the pre-drive check (card 16) read through receiver seams that `armAt` fills, with nil meaning no check, as the owner seams already do, so hand-populated receivers in the existing untagged tests, whose locations hold no seed, keep their behaviour.

### Decision: windows-relist-in-the-hook

D4's Windows re-list after the lock takes runs inside the caller's `AfterEnd` hook, after batten's locks are taken and before the strand stop, so `pairteardown` stays platform-neutral and the refusal releases the locks the hook took.

## PATTERN changes

- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended; a batten cancel's phase instead ends the owned processes and strands first and waits for the run lock after, and its removal leaves the board claim to the cancel, which clears it.
