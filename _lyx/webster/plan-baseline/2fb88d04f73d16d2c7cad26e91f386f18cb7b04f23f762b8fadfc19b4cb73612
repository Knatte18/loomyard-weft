---
format: 5
approved: true
language: go
---

# Plan: regression baseline and verify

Let a hub run a regression suite around a task: `Drift-Baseline` records the task's regression scope and captures the suite's profiles on the branch point, `Drift-Await` holds `Webster` until that capture passed, and `Drift-Verify` compares after the Code perch and spawns one strong agent to judge, accept and fix drift.
A new `internal/driftengine` owns the hub-wide `drift` config, the records, the capture and compare runs and the three producers; `internal/driftcli` is the `lyx drift` subtree; existing code gains a `capture` process role, breakaway-admitting Windows jobs, a teardown step for the capture's job, the activity reading, the `drift-verify` gate key and the recipe rows, every one behind `entry_gate: drift.enabled`.

## Card Index

1 — drift-config — the driftengine package with its hub-wide drift config module and paths
2 — drift-records — the scope record, capture record, counted-capture predicate and run counter
3 — drift-verify-gate-key — the drift-verify gate key and its mirrors
4 — job-breakaway — Windows step and loop jobs admit a breakaway start
5 — capture-run — the capture child's run, the side capture, the detached spawn and the lock probe
6 — compare-run — the compare run under its marker and the skip list
7 — drift-verbs-cli — the lyx drift verbs' rules and the driftcli subtree
8 — drift-stencils-tables — the two agent stencils and their seat tables
9 — drift-baseline-producer — the shared Deps and the Drift-Baseline producer
10 — drift-await-producer — the Drift-Await producer
11 — drift-verify-producer — the Drift-Verify producer and its decision halts
12 — drift-activity — the capture lock and compare marker read as loop activity
13 — capture-teardown — pair teardown ends the capture's named job
14 — drift-recipe-rows — the rows in the loom and darn recipes, their registry entries and wiring

## Shared Decisions

### Decision: engine and cli packages

- **Decision:** `internal/driftengine` holds every rule and producer and is handed absolute paths and seams; `internal/driftcli` resolves the worktree and builds the seams (PATTERN-cli-cobra's `<module>cli` over `<module>engine`). The decision record's "wired through `internal/cliwire`" is read as the hub CLI wiring every `<module>cli` does: `cliwire` serves only the standalone-capable CLIs (PATTERN-cliwire-sole-wiring), and `lyx drift` runs only inside a hub pair.
- **Rationale:** the engine stays testable on fakes at tier0; no new standalone mode is introduced.
- **Applies to:** all cards

### Decision: one Deps passthrough on Env

- **Decision:** the three rows read one `Env.Drift driftengine.Deps`, a whole-struct passthrough following `Env.Darn`; their settings are config keys read through `driftengine.LoadConfig`, and their on/off switch is their `entry_gate`, never a field or a producer switch (PATTERN-shed-recipe-registry).
- **Rationale:** the producers have behavior of their own whose seams a test substitutes one by one.
- **Applies to:** 9-drift-baseline-producer, 10-drift-await-producer, 11-drift-verify-producer, 14-drift-recipe-rows

### Decision: run status decides the caller

- **Decision:** a `lyx drift` verb tells the row's live agent session from a caller at a halt by the run status at `Drift-Verify` and the agent-session mark in `runs.json`: `running` with the mark open is the agent session, `blocked` a halt, and a decision halt a `blocked` status whose error opens with `shedadapters.DecisionHaltPrefix`; `running` with the mark closed is the row's own mechanical work, where no caller is the agent.
- **Rationale:** the status and the mark are Go-written truths every verb can read, the mark opened by `Drift-Verify` at its agent's fresh spawn, kept by an entry that reattaches to the live agent, and closed by an entry that finds no live agent and at the agent's end; no token is handed to the agent.
- **Applies to:** 2-drift-records, 7-drift-verbs-cli, 11-drift-verify-producer

### Decision: the verify gate reuses the row verify gate

- **Decision:** `Drift-Verify`'s test gate is the `verify` gate `resolveRowGateSpec` builds over the row's `gate_key: drift-verify`, i.e. `loomshed.NewVerifyGate` through `verifytree.Verify`, wrapped to run only once the session committed code; the dirty-tree and compare entries around it are driftengine's own.
- **Rationale:** no new verify site file, so `cmd/lyx/verifiedtree_test.go`'s site list is unchanged; the gate shares no pass with another key.
- **Applies to:** 11-drift-verify-producer, 14-drift-recipe-rows

### Decision: Windows job behavior proved outside this hub's gates

- **Decision:** the breakaway limit (4-job-breakaway) and the capture job's termination (13-capture-teardown) run only on Windows; this hub's gates run Linux, so their tier0 tests pin the seams and the call order, the cross-compile test builds the Windows files, and the end-to-end proof is the sandbox hub's run under board note `csharp-sandbox-hub` (Scope, Out).
- **Rationale:** no gate of this hub can run a Windows job object.
- **Applies to:** 4-job-breakaway, 5-capture-run, 7-drift-verbs-cli, 13-capture-teardown

### Decision: a dirty capture halt names cleaning

- **Decision:** a capture refused on a dirty tree with HEAD on the branch point names restoring the listed paths to HEAD or stashing them, then, at `Drift-Baseline`, `lyx loom resume`, which captures again, and at a `Drift-Await` halt `lyx drift capture` again followed by `lyx loom resume`, beside `lyx drift defer` followed by `lyx loom resume`; it names neither removing them nor committing them, which D3 lists.
- **Rationale:** the branch point is HEAD's merge base with the parent, so a commit moves HEAD off it and the resume halts naming only `lyx drift defer`; removing a tracked path leaves a tracked deletion, still dirty; a resumed `Drift-Await` only reads the records, so only a new capture progresses from its halt; PATTERN-refusal-way-forward requires a named way forward to progress, and that constraint wins over D3's wording.
- **Applies to:** 5-capture-run, 9-drift-baseline-producer

### Decision: seat outputs, segments and timeouts

- **Decision:** the baseline seat's only output is `baseline.md` and the verify seat's `verify-report.md`, since a fresh spawn archives its outputs and `scope.json` is Go-written; the seats take the `plan` and `review` segment colors; their session timeouts are package constants, since the decision record declares no timeout key.
- **Rationale:** a new segment needs a reed config key the decision record does not license, and archiving the scope record would lose the recorded scope.
- **Applies to:** 8-drift-stencils-tables
