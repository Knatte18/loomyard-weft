---
verdict: APPROVED
---

## Scope against the board entry

The decision record covers every Scope point of `engine-registry`'s board entry: `Spec.Engine` and the `engine: claude` key (D3), the engine set with its refusal naming the registered engines (D2), one `Runner` over the set with per-run engines from the record (D4), the run record's `engine` with an absent value read as the default (D5), per-run dispatch in the cross-run readers (D6), one registry constructor and the orch resume context through a capability (D1, D9), fork audit as an optional capability (D7), `Spec.Requires` checked at `Start` with a way forward (D8), `InstructionFiles()` (D10), `shuttleengine.ModelChoice` (D11), the `modelspec` engine names pinned to the registry (D12) and the PATTERN rule with its seam test (D13).

Departures from the entry's wording, none of which is a scope deviation:

- `ModelSwitchSequence` is deleted rather than made optional (D7).
  The entry's goal is a narrower `Engine` interface, which deletion meets; the method has no production caller (`grep` finds only the interface, `claudeengine`'s implementation and `shuttlefake`), and no board entry plans a live model switch.
- The registry is a new package `internal/shuttleengines` rather than `cliwire` (D1).
  The entry lists `cliwire` as a planning agent's recommendation "not yet decided by the operator", so the choice was open.
- `Open` resolves its engine from the spec, and stop and the step-stop sweep resolve none (D4).
  `Open` has no record to read yet, and stop and the sweep consult no engine, so this narrows nothing the goal needs.
- The PATTERN rule amends `PATTERN-shuttle-provider-seam` instead of adding a new entry (D13); the rule is the one the entry asks for.

The Out list matches the entry's Out and the neighbouring entries' scopes: config key moves, the neutral guard table and per-provider `SessionNamer` stay with `engine-config-seams`; filling `Spec.Engine` from aliases, each Claude-only role's `Requires` and Merriam's instruction files stay with `engine-model-choice`; tokencount and prices stay with `engine-usage-readers`; messaging stays with creel; anything Codex stays in note `codex-engine`.
The only `Requires` this task sets is the `ForkSubagents` implication, which is part of the mechanism the entry asks for, not a role declaration.

## Overlap with runs in flight

`lyx loom peers` gives `own_row` 3 and lists no parent commits.

- `creel-core` (row 9): its D17 rewrites the same `Runner.Send` and `Runner.Interrupt` record lookup so they work on a strand with no run record, and its delivery adapter adds one `claudeengine` construction site and constructs a shuttle `Runner`.
  The designs agree: creel's D17 drives a record-less strand with the runner's default engine, and this run's D4 gives a strand with no run record the set's default engine.
  Both records name the other and say the second to land adapts (this run's R1, creel's open risks), so this is not a conflict.
  Landing order: `creel-core` is further along (row 9 against 3) and lands first; this run takes its changes at Merge-In, routing creel's `Runner` construction and its `claudeengine` site through the registry, as R1 already plans.
- `webster-master-no-code` (row 5): both edit `internal/websterengine/doc.go` and `docs/overview.md`, at different places (the Master's read set against the engine-set and fork-audit wiring and the module tree).
  Text-only overlap. Landing order: `webster-master-no-code` lands first (row 5 against 3).
- `viewer-off-shell` (row 10): reed's focus rules in `internal/reedengine`; this run touches only `reedcli`'s watchdog call to `SessionNamer`.
  No overlap in behaviour. Landing order, were it to matter: `viewer-off-shell` lands first.
- `report-timing` (row 9): run-record reading in `reportengine`, shed caller labels and moved packages; no shared behaviour or PATTERN entry.

Shared bookkeeping: a possible `cmd/lyx/testdata/test-budget.yaml` row for `internal/shuttleengines` and the `docs/overview.md` module-tree line.
