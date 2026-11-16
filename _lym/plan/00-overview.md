---
format: 5
approved: false
language: go
---

# Plan: One run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked

A new leaf package `internal/runfsm` declares D1's run states, their order and the act rule.
A reader in `internal/pairteardown` derives a run's state from tracked records alone and asks the act rule for every caller.
The batten record carries the cancel intent with its finish and the run's pause intent, committed and pushed with today's pushes.
A cancelled run keeps its record as `cancelled` until its pair goes, and every removal of the pair deletes it.
The pull-request record and the fixer attempt counts move into the pair's tracked records.
The dobby's pass, `lym batten run`, `lym loom resume` and `lym tender resume` ask the act rule first.
The owner, hold, owed-push and code-tip inputs of the act rule are passed in by callers; this part fills the owner and the pause coverage, and later parts fill the rest.

## Card Index

1 — lym-files-cache-rule — PATTERN-lym-files: everything under `.lym` is a cache a fresh machine starts without
2 — runfsm-package — the leaf package with the states, their order, `Derive` and the act rule
3 — shed-cancelled-state — the shed state `cancelled` and the cancel and pause intent fields of the status file
4 — shedrun-run-record-writer — one writer for the run's tracked record and the cancel's local lock path
5 — fabric-origin-run-reads — fabric reads a task's records branch and origin's parent-side merge, pair or no pair
6 — run-reader-and-act-ask — `ReadRun`, `AskAct`, `ActRefusal` and `PublishBattenRecord` in pairteardown
7 — pull-request-record-tracked — the pull-request record moves into the run's records and every write is pushed
8 — fixer-attempts-tracked — the fixer attempt counts move into loom's durable directory
9 — cancel-intent-replaces-marker — the cancel records and pushes its intent before it ends anything; the marker goes
10 — cancel-finish-and-removal — the finish keeps the record as cancelled; a pair's removal deletes it; cleanup repairs leftovers
11 — dobby-asks-the-act-rule — dobby passes only a run the act rule lets act and hands an owned-elsewhere run over
12 — batten-run-asks-the-act-rule — `lym batten run` asks the act rule before its pre-run
13 — loom-resume-asks-the-act-rule — `lym loom resume` asks the act rule before it wakes a driver
14 — tender-run-pause-intent — `lym tender pause` and `resume` of a pair write the run's pause intent on origin

## Shared Decisions

- The run reader sits in `internal/pairteardown`, beside its batten-record and landing reads, because the act rule's callers all import it.
  Those callers are `battencli`, `loomcli` and `tendercli`, and `battencli` imports `tendercli`, so the reader cannot live in `battencli`.
  `runfsm` stays a standard-library leaf: callers turn records into its plain facts, so it holds no row name or shed state literal.
- `internal/fabricengine` gains two reads (card 5), a package the record's scope list does not name.
  D1 decides a landing's parent-side merge from origin's fetched parent branch and reads records from origin's branch when this machine has no pair;
  `PATTERN-fabric-boundary` puts both git reads in fabric.
  This answers the carry-over finding readrun-pastparentmerge-no-pair: the past-merge reading needs no local pair.
- The carry-over finding cancel-invisible-before-intent-committed is answered by card 9's order.
  The intent is written to the local record, committed and pushed before `EndForCancel` ends anything, so this machine's readers see it at once and other machines see it before any process ends.
  Offline, D3's bound applies.
- "Today's pushes" means `fabricengine.PushPairAnchored` for the synchronous pushes and the spawned status push for the rest.
  An unreachable origin leaves the commit for the next push; D2's owed-push retry, and the act rule's wait on it, come with a later part.
- The act rule's hold, owed-push and code-tip inputs stay false in this part.
  `ActRefusal` already words their refusals so the later parts only fill the inputs.
- `lym batten run` lets a landed verdict through to its start guard (card 12): a done record is either a done motif run that a raddle start clears, or refused by the guard with its own way forward.
- Dobby sets the shed pause aside only for a paused run whose work already reached its parent (card 11), so such a run is still driven to its landing as `drivable` allows today.
- The board's `<state> · <row>` display keeps its writers in this part.
  The only board reads the FSM makes are queued's empty status and landed's done, which `ReadRun` reads.
- The origin-owed marker of `internal/battencli/resumereconcile.go` stays, as the record's Delivers line says, until the part that retires it.
- A landing already in flight when card 8 lands gets one row entry's fixer cap again, since its old count sat in the scratch directory.

## PATTERN changes

- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
