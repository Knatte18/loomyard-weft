---
format: 5
approved: false
language: go
---

# Plan: One run FSM over tracked records, with cancel and pause intents, the PR record and fixer counts tracked

This part adds `internal/runfsm` (new): a run's lifecycle states, their order and the act rule, derived from tracked records alone (D1).
The dobby's pass, `lym batten run`, `lym loom resume` and `lym tender resume` ask it before they act.
The cancel intent with its finish and the run pause intent move into the batten record, committed and pushed with today's pushes; a cancelled run's record is deleted with its pair.
The pull-request record and the fixer attempt counts move into the pair's records (D3).
The owner, hold, owed-push and code-tip inputs are fields callers fill; this part fills the owner and pause inputs, and later parts fill the rest.

## Card Index

1 — shedengine-cancelled-state — Shed states gain `cancelled` for a finished cancel
2 — runfsm-core — runfsm's states, order, Derive and act rule, pure and tier0-tested
3 — runfsm-reader-batten-run — runfsm reads a run from tracked records; `lym batten run` asks the act rule
4 — loom-resume-asks-act-rule — `lym loom resume` asks the act rule first
5 — dobby-pass-asks-act-rule — the dobby's pass asks the act rule before it spawns
6 — cancel-intent-replaces-marker — the cancel intent, pushed in the batten record, replaces the cancel marker
7 — cancel-finish-keeps-record — a cancel writes and pushes its finish and keeps the record unless it removes the pair
8 — cleanup-cancelled-record — `lym batten cleanup` removes a finished cancel's record once its pair is gone
9 — teardown-deletes-cancelled-record — removing a cancelled run's pair deletes its batten record
10 — tender-pause-intent — `lym tender pause` and `resume` keep the run's pause intent on origin; resume asks the act rule
11 — pull-request-record-tracked — the pull-request record lives in the pair's records and is pushed on every write
12 — fixer-attempts-tracked — fixer attempt counts live in the run's records and ride its transition commits
13 — lym-files-cache — PATTERN-lym-files reads `.lym` as a cache a fresh machine starts without

## Shared Decisions

- **runfsm's imports.** The record calls runfsm a leaf: nothing it imports depends on it, and it imports no CLI module or run-driving engine.
  Its core (card 2) imports only the standard library; its reader (card 3) imports `lymcwd`, `shedrun`, `shedengine`, `boardengine`, `hubgeom`, `state` and `fabricengine`, since `tendercli`, which `battencli` imports, needs the same reader and so the reader cannot live in `battencli`.
  runfsm is not added to PATTERN-leaf-packages, whose entries pin a closed import set; no PATTERN change is declared for it.
- **Card order and PATTERN-test-only-code.** runfsm joins the production graph in card 3, whose caller reaches every function cards 2 and 3 create; each writer is created on the card of its first caller (`UpdateIntents`, `PublishIntents` and `ReadIntents` on card 6, `FinishCancel` on card 7).
- **Intents beside the shed keys.** The cancel and pause intents are the `cancel_intent` and `pause_intent` keys of the prime's `status.json`.
  shedengine does not declare them, and every Shed persist goes through `state.UpdateJSON`, which keeps undeclared keys, so Shed carries them through without knowing them.
  runfsm alone declares and writes them.
- **Today's pushes.** An intent or a finish is pushed synchronously with `fabricengine.PushPairAnchored` before its verb returns.
  With origin unreachable it stays committed here and the verb names its re-run; D2's retry of an owed push is a later part's.
- **Past the parent-side merge.** Read from the remote-tracking parent and task refs as this machine last fetched them, so it needs no pair; under the push skip the local parent branch decides it.
  `PastParentSideMerge` ignores the states' precedence, so a shed-paused run past its merge still finishes unless a pause covers it, as the dobby's reached-parent drive does today.
- **The cancel lock stays local and comes first.** The cancel takes its lock before it writes its intent, so this machine's driving verbs see the cancel in the window before the intent is committed; other machines see `cancelling` once the intent is pushed.
- **Inputs left at zero.** The hold, code-tip and owed-push inputs stay at their neutral zero values in this part's callers; the origin-owed marker of `internal/battencli/resumereconcile.go` stays until a later part retires it.
- **Board status.** The board's `<state> · <row>` status keeps being written as today; runfsm reads only its empty value (`queued`) and `done` (`landed`).

## PATTERN changes

- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
