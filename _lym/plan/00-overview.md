---
format: 5
approved: true
language: go
---

# Plan: Adopt runs as a detached, step-recorded worker that pauses every adopted run

`lym adopt` stops running its takeover in the caller's process.
A hidden `lym adopt worker` runs D3's steps in their new order — reconcile, pairs, pause, owner, deploy, beam, check, done — and records each in `adopt.json` under the hub's adopt scratch directory, so a killed worker resumes and `lym beam status` shows it.
The caller validates, spawns the worker detached and follows the record.
Every adopted run gets an adopt pause that only the operator lifts, the owner write moves before the deploy, `--no-orch` keeps the orch down, and a green adopt check only reports.

## Card Index

1 — adopt-pause-caller — add `pauserecord.AdoptCaller` and read it as held everywhere the adopt hold was read
2 — beam-up-skip-orch — let `orchcli.BeamUp` start the dobby without the orch
3 — adopt-record — add `internal/adoptrecord`: the adopt record, its path, lock and lock-free read
4 — green-end-reports — the green adopt check only removes the fix task and resets the try count
5 — run-report-paused-by-adopt — replace the held run states with `paused_by_adopt`
6 — adopt-step-order — restructure adopt into D3's ordered steps with the adopt pause and the owner write before the deploy
7 — adopt-worker — add the hidden worker verb that runs the steps under `adopt.lock`, records them and resumes
8 — adopt-caller-follow — `lym adopt` spawns the worker and follows the record; `--no-orch` and `--discard`
9 — beam-status-adopt — `lym beam status` shows an unfinished adopt
10 — adopt-docs-and-pattern — the `PATTERN-dobby-runs` change and the adopt package doc

## Shared Decisions

- The decision record names a D4 (D3's held-back paragraph says "so D4 still covers the held-back subject") but holds no D4 section, so no card plans one.
  Its Constraints and Auto-mode assumptions about the dobby's Bolt pull and the old owner's archive have no Decision to carry, and the plan carries none.
- The adopt pause reaches the dobby through `pauserecord.Coverage.Held`, which card 1 sets for `AdoptCaller` as well as the legacy `HoldCaller`.
  The dobby's held treatment then applies unchanged, so dobbyengine changes only in comments and its doc.
- Steps up to `owner` run in one process: a resume whose first step not done is `owner` or earlier restarts at `reconcile`.
  So only the inputs of `deploy` and later persist in the record's `Inputs`, together with `Paused`, which the `pause` step reads on a re-run.
- Each step's envelope part is the JSON of the outcome fields it set;
  the end envelope merges the done steps' parts in step order, then the part a stopped step gathered before its error.
- `adoptrecord` holds the record, its path, its lock path and its lock-free read; `adoptcli` alone takes `adopt.lock`.
- `adoptrecord.Request.Command` is the re-run text of every refusal and way forward that the record, the worker, the follow and the stop add;
  the existing reentry text (`reentry`) that ends the takeover's own refusals and warnings, the reconcile warnings included, stays.

## PATTERN changes

- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt pause lifts only by hand.
