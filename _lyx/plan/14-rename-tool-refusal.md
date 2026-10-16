# Card 14 — rename-tool-refusal

**Create:**
- `internal/websterengine#ErrRenameToolMissing`
- `internal/websterengine#renameToolRefusal`
- `internal/webstercli#TestRenameToolMissingEnvelope`

**Edit:**
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/websterengine_test#TestBeginBatch_Refusals`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#SymbolRenamePairs`
- `internal/websterengine_test#newRecoverFixture`

**Intent:**
D1.8: `begin-batch` and `recover-batch` refuse a batch holding a symbol `Rename` pair when the hub's `rename_tools` is empty, because a standalone run has no plan gate and a list emptied after the plan gate would otherwise fail the fork and both recoveries.

- `renameToolRefusal` returns nil when `renameTools` is non-empty or `planparser.SymbolRenamePairs(plan, batch.Cards)` is empty; otherwise an error wrapping `ErrRenameToolMissing` naming the batch, the first symbol pair and its card, and "way forward: set rename_tools with \"lyx config language\" from the prime, or recast the card, then re-run `lyx webster <verb> NN`".
  A batch holding only file pairs proceeds.
- `BeginBatch` calls it right after `PlanEditError`, over the batch `findBatch` finds, before the plan re-resolution or any mutation; an unknown batch falls through to the existing `findBatch` refusal.
  `recoverSpawn` calls it before it reads HEAD, so nothing is spawned or recorded.
  The input that reaches it: a plan that passes `planparser.Validate` with a card whose `Rename` pair has a member-glyph old side and a `plan:` handle new side, under an empty `RenameTools`.
- `beginBatchCmd` and `recoverBatchCmd` map `errors.Is(err, websterengine.ErrRenameToolMissing)` to `output.ErrFields(out, err.Error(), map[string]any{"rename_tool_missing": true})`.
- `webster-template-master.md`'s failure ladder gains the rung: `begin-batch <NN>` or `recover-batch <NN>` refuses with `{"rename_tool_missing": true}` → write `outcome: stuck` to the outcome file with a `stuck_reason` quoting the refusal's message, and stop; the hub's rename tool is the operator's to set.
- `refusal-spec.md` `## webster` gains the row: rename tool missing; trigger as above; correctness halt; way forward as the message.

Tests:
`TestBeginBatch_Refusals` gains rows: a batch with a symbol pair under an empty list refuses matching `ErrRenameToolMissing` and records no batch; the same batch with a non-empty list and a batch with only a file pair under an empty list each return an error that does not match `ErrRenameToolMissing` or no error, since the call goes on to the plan re-resolution, which may refuse the fixture's glyph for its own reason.
`TestRecoverSpawnOrAttach`, beside its existing refusal rows, gains two rows whose setup gives the fixture's batch card and plan card a symbol `Rename` pair: under an empty `fx.Deps.RenameTools` the refusal matches `ErrRenameToolMissing`, the prior record is unchanged and `fx.Engine.PrepareCalls` stays zero; with only a file pair under an empty list the recovery spawns.
`TestRenameToolMissingEnvelope` runs `begin-batch` and `recover-batch` through the CLI over such a plan with no `language.yaml` list and asserts each exits non-zero with `"rename_tool_missing":true`; `TestMasterStencilCoversEverySingleFlagRefusal` then holds the new rung by its existing scan.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/webstercli` row rises by one.

**ImpactSummary:** two verbs gain one refusal before any mutation; a batch without a symbol pair, or a hub with a tool listed, is unaffected.
