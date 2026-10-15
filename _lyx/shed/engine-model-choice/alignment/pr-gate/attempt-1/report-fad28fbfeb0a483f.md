---
verdict: APPROVED
---

## Alignment at PR-Gate: engine-model-choice

Pull request 665 at `7a6edad38ded99385d3390d9082d0167dff91f82`, judged against the board entry's goal and decisions D1 to D8 of the decision record.
Its merge base with `main` is `main`'s head `a44ef5e66`, so no parent commit lies between them, and `lyx loom peers` lists no parent commits.

### Goal and decisions

- D1: `shuttleengine.Spec` embeds `ModelChoice` and drops its own `Model`, `Effort`, `Version` and `Engine`; the field docs moved onto `ModelChoice`.
- D2: every changed resolver builds its choice through `NewModelChoice`.
  `PATTERN-model-choice-mapping` lands in `PATTERN.md` with its background file and the scan `TestModelChoiceMappingRule`, in the same commit (card 14).
- D3: `burlerengine.ModelChoice` is gone; burler, the Bouncer pair, seats, treadle's profile, alignment, the tackler and the driver carry `shuttleengine.ModelChoice`.
  `lyx burler run` builds a choice with an empty engine.
- D4: `Result.Engine` is added, and `round-N-usage.yaml`'s half and pass entries gain an omitted-when-empty `engine` key taken from the run's result.
- D5: Master and the loom driver require background wake beside implied forks.
  The Discussion and Plan leads require background wake when lenses are on.
  A table of more than one seat adds peer messaging to every seat.
  The cluster-fan reviewer's forks stay implied by `ForkSubagents`.
- D6: the wiring's `AuditEngineFrom` check is gone.
  `Run` checks both roles' engines after the plan-approved and `Batcher` gates and before `AcquireStateMutation`.
  `RecoverSpawnOrAttach` checks the recovery engine after the live-recovery return and ahead of `AcceptBatchFabricReference`, the stale-report archive and `StopStrand`.
  State records `masterEngine`, each batch's `engine` and `recoveryEngines`, carried forward with `RecoverySessions`.
  Sessions are read through `sessionAuditEngine`'s three-way lookup.
- D7: `MerriamBase` takes the instruction file names, which reach it lazily through `MasterInstructionFiles`.
- D8: `contracts/specs/refusal-spec.md` gains the webster and loom rows.
  `ModelKey.Wrap` names the caller's key, and `lyx burler run` names the default engine.

The orch spec only takes the D1 field move and keeps its Claude-only model, as the record's Out says.
No config key is renamed, and `codex` is not added to `modelspec` or the registry.

### Open risks carried into the record

- Discussion-Review `d6-wiring-check-verb-placement` and `d6-spawn-check-after-mutation`: the diff removes the wiring check and places both spawn checks ahead of every state mutation, as D6 states.
- Merge-Review `recordless-send-interrupt-ignore-strand-engine`: fixed in `a8f915680`.
  Recordless `Send` and `Interrupt` now resolve the engine from reed's strand provider tag, which `reedengine.StrandStatus` now carries.
  The goal calls for this, since a strand spawned on its role's engine would otherwise be driven through the default.
- The pre-change engine-less state entry and `creel-agents`' later revisit of the seat `peer-messaging` rows stay accepted open risks of the record.

### Bookkeeping files

- `cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttlecli` and `internal/shuttleengine` rows are raised by one each.
- `PATTERN.md`: a new index line for `PATTERN-model-choice-mapping`.
  To stay under the overview's size line, four one-liners (`PATTERN-pair-teardown`, `PATTERN-shed-driver`, `PATTERN-shuttle-provider-seam`, `PATTERN-guarded-interrupt`) are trimmed.
  Their dropped clauses still stand in each entry's background file, so no rule is changed.

### Overlaps and landing order

`own_row` is 22.

- `engine-usage-readers` (row 9, Plan-Burler-Review, no changed paths yet): both runs touch the engine's instruction-file names and the usage that `round-N-usage.yaml` records.
  Its decision record leaves `websterengine.MerriamBase` to this run, and its D8 mirrors this run's D7 for calibration.
  Its own open risk tells whichever run lands second to re-check that the two lists agree.
  The designs agree, so it does not escalate.
  Landing order: this run lands first, and `engine-usage-readers` takes its changes at Merge-In.
- `card-own-entry-edit` (row 5, Discussion-Burler-Review, no changed paths yet): both add a new `PATTERN.md` entry, each a different one.
  That is a new index line only, and the runs share no behaviour.
  Landing order: this run lands first, and `card-own-entry-edit` takes its changes at Merge-In.

No board entry is made obsolete.
The notes `loom-model-key-names`, `shed-arbiter-all`, `codex-engine` and `creel-agents` wait on this task and stay open.
