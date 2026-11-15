---
format: 5
approved: false
language: go
---

# Plan: one run FSM over tracked records, with cancel and pause intents

A new leaf package, `internal/runfsm`, derives every run state of D1 from tracked records alone and answers the act rule from inputs its callers pass in.
`pairteardown.ReadRun` is the one reader of those inputs.
The dobby's pass, `lym batten run`, `lym loom resume`, `lym tender resume <slug>` and `lym batten cancel` ask the rule before they act.
The cancel intent with its finish and the run pause intent move into the batten record, committed and pushed with today's pushes.
A cancelled run keeps its record, which every path that removes its pair deletes.
The pull-request record and the fixer attempt counts move into the pair's tracked records.
The origin-owed marker stays until the later part that retires it.

## Card Index

1 — shed-run-intents — add the cancel and pause intent fields and the cancelled state to the batten record
2 — shedrun-intent-writers — write and clear the intents in the batten record under its status lock
3 — runfsm — the run states, their order, Derive, the act rule, Judge and the board display state
4 — fabric-origin-reads — read origin's fetched parent branch and a child record from origin
5 — pairteardown-read-run — read a run's FSM records and act inputs in one place
6 — board-display-from-fsm — write the board's run status from the FSM state
7 — cancel-intent — the cancel records its intent and finish, keeps or deletes the record, and asks the act rule
8 — cancel-readers-intent — every cancel-marker reader reads the record's intent; the marker file goes
9 — cancelled-record-cleanup — cleanup and fabric remove delete a cancelled record whose pair is gone
10 — batten-run-act — a hand batten run asks the act rule
11 — dobby-pass-act — the dobby's pass asks the act rule and reads the FSM state
12 — loom-resume-act — loom resume asks the act rule
13 — shared-bolt-pull — move the Bolt pull into boardengine for tendercli
14 — tender-pause-intent — tender pause and resume write and clear the pause intent, refuse an owner elsewhere and ask the act rule
15 — pr-record-tracked — the pull-request record lives in the tracked records and each write commits and pushes
16 — fixer-attempts-tracked — the fixer attempt counts live in the tracked records
17 — lym-files-pattern — PATTERN-lym-files: .lym holds only caches

## Shared Decisions

### Decision: runfsm-imports

- **Decision:** `internal/runfsm` imports only the standard library and `internal/shedengine`, for the record type, and does no I/O; `pattern/PATTERN-leaf-packages.md` records it.
  Callers fill the row facts (`BattenStarted`, `ChildLanding`) from the recipes, so the package names no row.
- **Rationale:** D1 makes it a leaf; taking `shedengine.Status` whole lets the intents of card 1 reach it unconverted.
- **Applies to:** cards 3, 5

### Decision: shed-pause-aside

- **Decision:** `lym batten run`, `lym batten step`, `lym loom start` and `lym loom resume` set aside the shed pause of the record they resume (`runfsm.Aside.ShedPause`).
  `lym batten cancel` and `lym tender resume <slug>` set it aside too, since neither drives the run: the cancel ends a run whatever its state, and the resume lifts only its pause entry.
- **Rationale:** D1 reads a shed state `paused` as `paused` and has the act rule wait on it; the verb that resumes that shed pause would otherwise refuse its own resume, leaving no way forward.
  The paused refusal names `lym batten run <slug>`, the verb that resumes it.
  Every other caller, the dobby's pass included, waits on it.
- **Applies to:** cards 3, 7, 10, 12, 14

### Decision: owed-push-aside

- **Decision:** `lym batten cancel`, `lym tender pause <slug>` and `lym tender resume <slug>` set the owed-push wait aside (`runfsm.Aside.PushOwed`).
- **Rationale:** D3 has each of them act at once with origin unreachable and owe its push, so the later part that fills `PushOwedOffline` must not make them refuse.
- **Applies to:** cards 3, 7, 14

### Decision: paused-before-landing

- **Decision:** The dobby no longer gives a pass to a batten-paused run whose work reached its parent; the act rule waits it until `lym batten run` resumes it.
- **Rationale:** D1's order puts `paused` before `landing`, and only a `landing` run past its merge acts under a hold.
- **Applies to:** cards 10, 11

### Decision: reader-in-pairteardown

- **Decision:** `pairteardown.ReadRun` is the one reader of the FSM's inputs, and `boardengine.PullBolt` the one Bolt pull.
- **Rationale:** `tendercli` needs both and cannot import `battencli`, which imports it; `pairteardown` already holds the batten-record and owner-pause readers.
- **Applies to:** cards 5, 7, 10, 11, 12, 13, 14

### Decision: fabric-origin-reads

- **Decision:** Card 4 adds two reads to `internal/fabricengine`, a package the record's scope list does not name.
- **Rationale:** D1 decides the parent-side merge from origin's fetched parent branch and reads a record from origin's branch when this machine has no pair; `PATTERN-fabric-boundary` puts every git read of warp or weft there.
- **Applies to:** cards 4, 5

### Decision: later-part-inputs

- **Decision:** `runfsm.Inputs.PushOwedOffline`, `CodeTipMissing` and `Held` stay false in this part; D2, D4 and D7 fill them.
  The adopt pause of today is a live pause entry of the slug's own scope, so `PauseCovers` already waits on it and `lym tender resume <slug>` lifts it, as `PATTERN-dobby-runs` says.
  D11's dobby teardown, D6's catch-up and the origin-owed marker's retirement are not this part's.
- **Rationale:** The record's Delivers line: the owner, hold, owed-push and code-tip inputs are passed in by callers, and the origin-owed marker stays until bolt-holds-catch-up.
  Reading the adopt pause as a hold would have `lym tender resume <slug>` refuse the very entry it lifts.
- **Applies to:** cards 3, 5

## PATTERN changes

- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
