---
format: 5
approved: false
language: go
---

# Plan: push retries, never halts

lym stops runs and mails the orch when a push or a GitHub call fails on a flaky network.
After this plan lym and its agents only commit: one background pusher brings every branch lym owns to origin with a quiet, adaptive retry,
a landing merges locally under the parent lock and hands its push to the parent target's pusher, the Go driver waits out a network stop,
and only a refusal that retrying cannot fix stops anything.
The cards run in order: timing config and the driver's wait first, then the ledger and the pusher's targets, then the landing, the teardown and the catch-up on top of them, and the notices, the bar and the docs last.

## Card Index

1 — push-timing-config — add fabric.push_interval_s, push_ceiling_s and push_notice_after_min with a floored accessor
2 — driver-network-wait — re-step a network stop without a cap, waiting from 15 s to the ceiling, under a wait record
3 — driver-wait-readers — read a waiting run as waiting in the halt note, loom status and the dobby's run watch
4 — push-ledger — add the leaf package holding the owed ledger, the landing entries and the hub's clear time
5 — pusher-interval — push once per interval in force, doubling on failure, kept in the ledger, with no notice
6 — pusher-target-kinds — add the board and landing targets, the pusher command line and the dobby's restarts
7 — spawns-start-the-pusher — make every detached push spawn start or wake the pusher instead of pushing
8 — owed-code-commits — read a pair owed whenever its code branch is ahead of origin
9 — records-hold — keep a landing entry's held records commit off origin in every synchronous run push
10 — parent-push-git — add the prime code branch's fast-forward push and its refusal records
11 — parent-pusher — push, settle, drop and hold back landings on the parent target with its owner reads
12 — finalize-local-landing — land locally, record the landing entry and return done; owner reads wait for origin
13 — finalize-pr-owed-check — stop a GitHub landing as git-transport while its parent owes a landing push
14 — merge-in-leftovers — remove a deleted directory's ignored leftovers from a persisted pre-merge base
15 — run-reader-past-merge — read past-the-merge from the local parent while origin's lacks the work
16 — prime-pull-owed-tips — read owed landing tips as level on a prime pull and Land's unreachable pull as git-transport
17 — catch-up-hands-to-pusher — hand the catch-up's owed push to the pusher and start the run
18 — reset-hand-commit-guard — refuse a reset past an origin commit no local branch or reflog reaches
19 — teardown-hands-to-dobby — wake the pusher from Worktree-Teardown, refuse teardown while a landing stands
20 — dobby-push-summary — post one owed summary past the threshold and one all-clear from the prime's dobby
21 — sync-status-bar — add lym fabric sync-status and the bar's sync field
22 — edit-directives — tell agents a failed push needs no action
23 — pattern-docs — rewrite the PATTERN-pair-remote line and the backgrounds the change falsifies

## Shared Decisions

### Decision: one leaf ledger package

- **Decision:** the owed ledger and the landing entries live in the new `internal/pushledger`, which imports only `state`, `lock`, `lymdirs`, `fabriclayout` and the standard library.
- **Rationale:** `fabricengine`, `boardengine`, `pairteardown`, `runpusher`, `dobbyengine` and `fabriccli` all read it, and `runpusher` imports `fabricengine`, so the ledger cannot live in `runpusher` without a cycle.
- **Applies to:** cards 4 to 21

### Decision: a spawn starts the pusher through its command line

- **Decision:** a detached spawn launches `lym loom pusher --push-now` for its target, built by `fabriclayout.PusherCommand`; the pusher verb sets the push-now mark and exits at a held lock.
- **Rationale:** `fabricengine` and `boardengine` cannot import `runpusher`, and a second process that only wakes the running pusher keeps one pusher per target.
- **Applies to:** cards 6, 7 and 11

### Decision: a landing entry is the hold and the pending done write

- **Decision:** a run's landing entry on its parent target names the landed tip and the records commit it holds; the pending `done` write lasts exactly as long as the entry.
  The records hold lasts as long as the entry, and past a dropped landing it lasts until the pair goes: the drop moves the held commit into the pair target's own entry as `HeldRecords`, which `RemovePair` removes with the pair (D5).
- **Rationale:** one record answers every reader, the pushes of card 9, the teardowns of card 19, the pulls of card 16 and the settles of card 11, so no two can disagree.

### Decision: a landing commit carries its run in a trailer

- **Decision:** Finalize's landing commit carries the trailer `Lym-Landing: <slug>` (`fabriclayout.LandingTrailerKey`), so `fabricengine.LandingCommits` finds every landing commit on a parent's first-parent chain past origin, recorded or not.
- **Rationale:** D5 refuses a parent push whose tip carries a landing commit no entry records, and re-reads each entry's tip after a rebase as the local parent commit that merges its run; a trailer is the one mark both reads can find on the prime and on a beam parent alike.
- **Applies to:** cards 10, 11 and 12
- **Applies to:** cards 4, 9, 11, 12, 15, 16, 17 and 19

### Decision: the change description settles D12

- **Decision:** no card changes code for D12. The change description names issue #218 (the code-tip hold `internal/runfsm/act.go` no longer applies to a pair standing here) and issue #254 (the loomcli status-push loop) as settled by `async-run-pusher`, and says #254's ask to bound the owed retry is replaced by the quiet retry, whose outage D3's summary and D4's field make visible.
- **Rationale:** D12 asks that what the pusher already settles be named and dropped.
- **Applies to:** the change description

## PATTERN changes

- `PATTERN-pair-remote` — changes: Detached pushes of lym's branches: one background pusher per target, records before code, owed pushes retried quietly; a push whose result its caller reads stays synchronous; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
