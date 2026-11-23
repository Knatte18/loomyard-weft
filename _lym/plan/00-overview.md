---
format: 5
approved: false
language: go
---

# Plan: later run transitions still wait for a records push to origin

No batten or loom transition waits for a push to origin any more.
Every transition and every driver-stop records commit commits locally and starts its records worktree's pusher,
a detached process that pushes the records side and then the code side in order, retries an owed push without recording it,
stops on a refused push, runs loom's pre-push reconcile before each push of a pair, and hands a taken-over run over.
The dobby's tick and a resume restart it, Worktree-Teardown pushes the pair to completion before it removes it,
and the PATTERN entries and docs state the new rule.

## Card Index

1 — fabric-owed-unrecorded — a pair push mode that returns owed failures without writing their records
2 — fabric-pusher-reads — local reads of ahead-of-origin, a standing refused push and tips held on origin
3 — fabric-prime-takeover — a rejected prime push reads every batten run's takeover and leaves handed-over runs out
4 — pairteardown-prime-handover — PushRun hands each taken-over batten run over, then pushes the rest
5 — runpusher-package — the pusher: start conditions, spawn, push loop, owed backoff and exit protocol
6 — loom-pusher-verb — the hidden `lym loom pusher` verb wiring the push, the pre-push guard and the notice
7 — transitions-start-pusher — transitions and records commits commit and start the pusher, never push
8 — restarts-start-pusher — the dobby's tick and a resume start the pusher
9 — runread-local-code-tip — a run whose own pair holds its missing code tip is not held
10 — teardown-pushes-first — Worktree-Teardown pushes the pair with the prime lock released before removing it
11 — pattern-and-docs — the PATTERN entries, backgrounds, glossary and overview doc state the Decisions

## Shared Decisions

### Decision: the pusher's home

- **Decision:** the pusher's start and loop live in a new package, `internal/runpusher`, with every git read in `internal/fabricengine`.
  The process runs as a hidden `lym loom pusher` verb, since a pair's push passes loomcli's `prePushCheck` (D7).
  It starts in the hub's Bolt directory and is registered under `runproc.HubOwner` with role `runproc.RolePush`.
- **Rationale:** batten, loom, the dobby and the resumes all start it, and none of them may import loomcli but loomcli itself;
  the verb only needs the worktree path, so its argv is the only coupling.
- **Applies to:** cards 5, 6, 7, 8 and 10

### Decision: the pause is the pusher's own read

- **Decision:** the pusher reads the pause before the guard on every push attempt and exits on it; `skipsCheck` keeps skipping on a pause.
- **Rationale:** the landing's pushes, out of scope, keep today's pause skip; only the pusher's push stops on a pause (D7).
- **Applies to:** cards 5 and 6

### Decision: handed-over batten commits are set aside like an ended run's

- **Decision:** a handed-over batten run joins the `EndedSet` the prime's replay reads, under its own value,
  so its commits go under local set-aside tags and the rest replay in order.
- **Rationale:** a batten commit stages only its own run's directory, which the ended-run set-aside already keys on.
- **Applies to:** card 3

### Decision: the out-of-scope record pushes keep their push

- **Decision:** batten's `commitStatusDeps` keeps `Push` and `pushRecordsOrOwe` for the cancel's record commit and the re-adopt's failed-stop commit;
  only the transition seam and `commitRunRecords` stop calling them.
- **Rationale:** the cancel intent is a tracked intent and the re-adopt is `lym adopt`'s own step, both outside the decision record's scope.
- **Applies to:** card 7

## PATTERN changes

- `PATTERN-run-fsm` — changes: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and the run acts at once, a background pusher bringing every commit to origin in order; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, from the worktree's background pusher, which no transition waits for; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read via hubgeom's `ParentOf` and Bolt's machine registry; a loom pusher reconciles first; after absence, a catch-up from origin first.
