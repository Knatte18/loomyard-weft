---
format: 5
approved: false
language: go
---

# Plan: fabric core — push classification, retries, origin reconcile and failed-push records move to the core

Part 2 of the fabric restructure (D9).
The push and origin concern's rules move into `internal/fabriccore` as phase and action-list rules over made-up snapshots, with retries as wait actions and the teardown as a seam-call action;
`internal/fabricengine` runs them through the foundation's executor loop, keeping the git reads, the locks, the live-pair probe, the error-to-class mapping and the guards at execution.
The concern's files stop importing `internal/shedrun` and `internal/shedengine`: run paths, the prime run lock and the status codec reach fabric through required seams one installer in `internal/fabriccli` puts in every process, and the status reader returns bytes.
The concern's tests shrink: rules become tier0 core rows, git effects and interleavings stay tier1 in fabric, and flows resting on the board's replay move to `internal/boardengine`.

## Card Index

1 — core-required-seams — the required-seam snapshot fact and its refusal rule in the core
2 — shed-status-adapter — shed's status codec verdict over a record's bytes
3 — required-seam-installer — fabric's required-seam setters, the fabriccli installer, its main.go and hubforge calls
4 — origin-reads-seams — origin status reads return bytes through the seams and refuse an unset seam
5 — failed-push-core — the failed-push record's merge, notice, way forward and filters move to the core
6 — push-lock-holder-core — the push lock holder clause and the coalescing progress test move to the core
7 — push-side-core — pushClassified runs as a core rule with wait and retry-seam actions
8 — pair-push-core — the pair push's records-side loop and the pusher's local reads move to the core
9 — origin-judge-core — ReconcileOrigin's judging runs as core phases under the engine's locks
10 — origin-ended-core — the removal of ended runs is a core rule with teardown seam calls
11 — origin-moves-core — the reconcile's move order and verdicts come from the core, guards stay at execution
12 — board-test-moves — tests resting on the board's replay move to internal/boardengine

## Shared Decisions

### Decision: what each part card's commit message carries

- **Decision:** every card that deletes, moves or keeps a fabric test lists in its commit message body each test and row it deleted and, for each test it kept or moved, the rule that test alone guards; a deleted assertion names the test or core row that now holds it (D5, added entry "The restructure cuts fabric's test code").
- **Rationale:** the record makes the change description the review's evidence that the concern's test code shrinks.
- **Applies to:** cards 4–12

### Decision: the retry bodies stay one seam-call action each

- **Decision:** `retryWeftPush`, `retryBoltPush` and `repushOverOriginContained` run as the one action `SeamRejectionRetry` names; card 7 moves the decisions over their outcomes, the settle loop and the classes, into the core.
  Their bodies are replays and the reset of an unnamed tip; the replay rules belong to the replay concern of part 5 (D9), and splitting them here would change that concern's files.
- **Rationale:** D9 keeps each part to its own concern's files; the seam-call action is D3's shape for an answer a rule branches on.
- **Applies to:** cards 7, 8

### Decision: fabric tests of this concern record no run and diverge no Bolt branch

- **Decision:** a fabric test that runs `ReconcileOrigin` or a records push records no run, asks no owner and diverges no Bolt branch, so it needs no Bolt replayer; a row that needs one rests on the board's ended set, owners or entry merge, and moves to `internal/boardengine` (D12).
- **Rationale:** the run ledger is read only through a registered replayer (`runLedger`), and only for those three cases; registering a stand-in replayer would feed fabric made-up data for another module (D12).
- **Applies to:** cards 9–12

### Decision: required seams are set only through their setters, sequentially

- **Decision:** a fabric test that unsets or sets a required seam builds its hub through `internal/hubforge` first, uses the seam's setter, restores it, and does not call `t.Parallel`; every other test takes the seams `internal/hubforge` installed once per test binary (D13).
- **Rationale:** Go runs a binary's parallel tests after its sequential ones, so no parallel test sees a seam change.
- **Applies to:** cards 4, 9, 11
