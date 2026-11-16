---
format: 5
approved: false
language: go
---

# Plan: one run FSM in internal/runfsm over tracked records, with cancel and pause intents, the PR record and fixer counts tracked

This part delivers D1 and D3 of the decision record.
A new package, `internal/runfsm`, derives every run state from tracked records alone and answers the act rule;
the dobby's pass, `lym batten run`, `lym loom resume` and `lym tender resume` ask it before they act.
`lym batten cancel` and `lym tender pause` record their intents in the batten record, committed and pushed;
a cancel keeps the record with its finish, and every removal of a cancelled run's pair deletes it.
The pull-request record and the fixer attempt counts move into the pair's tracked records.
The origin-owed marker stays until a later part (bolt-holds-catch-up) retires it.
The carried-over Plan-Review findings are answered in place:
`cancel-invisible-before-intent-committed` by card 9's order, intent pushed before anything ends;
`readrun-pastparentmerge-no-pair` by card 5's read of origin's fetched branches through the prime, with no pair needed.

## Card Index

1 — lym-files-cache-pattern — declare `.lym` a cache a fresh machine rebuilds or starts without
2 — shed-cancelled-state-intents — add the cancelled state and the cancel and pause intent fields to the shed status
3 — runfsm-states-derive — start internal/runfsm with D1's states, their order and Derive
4 — runfsm-act-rule — add the act rule with its inputs and asides
5 — runfsm-tracked-reads — read a run's records from the hub and origin, and the owner input
6 — dobby-asks-act-rule — the dobby's pass asks the act rule before it gives a run a pass
7 — batten-run-asks-act-rule — lym batten run refuses what the act rule waits on or hands over
8 — loom-resume-asks-act-rule — lym loom resume refuses what the act rule waits on or hands over
9 — cancel-intent-in-batten-record — the cancel records and pushes its intent and finish in the batten record
10 — cancel-marker-retire — delete the cancel marker and rename dobby's view of it
11 — cancelled-record-removal — fabric remove deletes a cancelled run's record with its pair; cleanup repairs crash leftovers
12 — pull-request-record-tracked — the pull-request record becomes tracked, and each write commits and pushes it
13 — fixer-attempts-tracked — the fixer attempt counts become tracked records committed with the transition
14 — tender-pause-intent — tender pause and resume record the pause intent, refuse another machine's run, and resume asks the act rule

## Shared Decisions

### Decision: intents-live-in-the-batten-record

- **Decision:** The cancel and pause intents are optional fields of `shedengine.Status`, so the batten record, the prime's `status.json` of the slug, holds them.
  Shed never sets or clears them, and each of its persists rewrites only its own fields through `internal/state`, so an intent written between two persists survives.
  A finished cancel sets the state `cancelled`, which no reader takes for `done`.
- **Rationale:** D3 puts the intents in the batten record; one tracked file keeps them with the run's state on every machine.
- **Applies to:** cards 2, 9, 11, 14

### Decision: runfsm-reads-tracked-records-through-fabric

- **Decision:** `runfsm` reads the board, the prime's batten record and the pair's record from disk with no lock and no `.lym` path.
  It reads origin's records branch, origin's task branch and origin's fetched parent branch only through `fabricengine`, and runs no git itself.
  Its writers write the batten record under the status lock, a local lock, and commit and push through `fabricengine`.
- **Rationale:** D1 derives the state from tracked records alone; `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary` keep git in `fabricengine`.
- **Applies to:** cards 3, 4, 5, 9, 14

### Decision: act-inputs-this-part-fills

- **Decision:** This part fills the act rule's owner input from the Bolt's owners through `hubgeom.ParentOf`, and its pause coverage from this machine's pause record.
  `Held`, `PushOwedOffline` and `CodeTipMissing` stay zero, read as no hold, no owed push and the code tip present, until the parts for D7, D2 and D4 fill them.
- **Rationale:** The record's Delivers line passes those inputs in from callers, and later parts own the records behind them.
- **Applies to:** cards 4, 6, 7, 8, 14

### Decision: dobby-pass-takes-the-run-verbs-shed-pause-aside

- **Decision:** The dobby's pass is a `lym batten run`, so the dobby asks the act rule with that verb's shed-pause aside,
  and sets it only for a run whose work already reached its parent, the one paused run today's planner drives.
  Any other shed-paused run waits, as today.
- **Rationale:** Without the aside, a run a cancelled pass left paused after its parent-side merge would wait for a hand run, where today the dobby finishes its landing.
- **Applies to:** card 6

### Decision: refusals-keep-their-exit-meaning

- **Decision:** An act-rule refusal is a `*runfsm.Refusal`; batten's pass exit maps a `hand_over` to the owned-elsewhere exit and a pause-coverage `wait` to the covered exit.
  `lym loom resume` keeps its covered and owned-elsewhere refusals and kinds ahead of the act rule.
- **Rationale:** The dobby reads those exits to owe passes and run its owner handling; a new generic failure would count as a failing pass.
- **Applies to:** cards 7, 8

### Decision: offline-intent-push-until-d2

- **Decision:** With origin unreachable, a cancel commits its intent without a push and names its rerun as the way forward, as D3 says.
  A slug pause or resume commits its intent and spawns today's detached push in place of D2's retry, and its report says the push is owed.
- **Rationale:** D2's retry belongs to a later part; the detached push is the existing way a local commit reaches origin later.
- **Applies to:** cards 9, 14

## PATTERN changes

- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
