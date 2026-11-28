```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal points of the amended board entry and the `Added after Discussion` entries.

**Decisions and the changes that build them**

- **D1** (jacquard recipe file, vocabulary, shedcli table, embed, loomrecipe map and lists). Built by:
  - C1: new `jacquard-recipe.yaml`. Its rows from Drift-Baseline on, and its top-level keys, are identical to the base tapestry file; the `diff` returned ROWS-IDENTICAL and TOPKEYS-IDENTICAL.
  - C2: `shedrun.RecipeJacquard` and `RecipeNames`. `OfferedRecipeNames` and `ValidateRecipe` derive from `RecipeNames`.
  - C3: shedcli `table.go`.
  - C4: `recipes.go` embed.
  - C5: loomrecipe `embeddedRecipes`, `RecipeEngines`, `BouncerRunSubdir` and `IsFinalizeRow`.
- **D2** (tapestry as the Motif and Jacquard blocks, entry Motif, row names unchanged, headers rewritten). Built by:
  - C6: `tapestry-recipe.yaml` rewritten as the two block rows with entry Motif.
  - C7: `motif-recipe.yaml` header names jacquard.
  - C8: `loomshed.NameJacquard` and its interrupt-policy entry.
- **D3** (Expand admits a last always-gated block row and refuses the other cases). Built by:
  - C9: `expand.go` `checkRunEndingBlock` and its doc.
  - C10: `expand_test.go` rows for admitting and refusing.
- **D4, engine and contract steps.** Built by:
  - C11: the shedrecipe registry entry, `LoomSeedEngine`, `Env.RecordSeed`, `entries_recordseed.go` with its test, `doc.go`, and the test fixtures.
  - C12: `loomshed/recordseed.go` and `recordwrite.go` stored-only mode, `NameRecordSeed` and its interrupt policy, `doc.go`, and `recordseed_test.go`.
  - C13: `loomcli/wiring.go` `recordSeedDeps` and the `EngineRow` read of the Discussion row.
  - C14: `testkit/envkit` `FullEnv.RecordSeed`.
  - C15: `loomcontract.go` `recordSourceRow`, `rowsOf` and the contract doc, with `build_test.go` rows.
  - C16: `loomrecipe.RecordEngine` returns RecordSeed, with `recipekeys_test.go` rows.
- **Added entry: jacquard stays one recipe; no plan or build recipe, pair handover or Worktree-Teardown skip.** Honoured: the diff adds none of these.
- **Added entries (arbiter and parent): the jacq nickname is written after landing.** Honoured: no file of the diff writes `heddle.json` or a nickname line.
- **Added entry: no motif-run clear; a Sketch goes only to raddle.** Honoured: the diff adds no batten motif-run clear.

**Changes forced by a traced change**

- C17: `docs/overview.md` lines for loomrecipe, shedcli and loom. The overview names these packages, and C1, C3 and C5 made those lines stale.
- C18: loomrecipe tests switched to `parseEmbeddedRecipe`, plus `setRowProducer`, and updates to the shape, progress, entry-number, discussion-approval, sequence and block tests. Forced by C6: tapestry's raw file no longer holds the rows, and Motif is now the first row. Two new walk tests also build D2's Delivers.
- C19: battencli `overview_test.go` stage numbers and the `wire_seams_test.go` nil-seam entries. Forced by C6, C11 and C12.
- C20: loomcli `specfill_test.go` entry and the `//lym:seam` lines in the wiring tests. Forced by C6 and C12.
- C21: envkit `envkit_test.go` segment sources switched to motif and jacquard. Forced by C6: raw tapestry no longer holds segments.
- C22: shedrecipe `registry_test.go` and `conformance_test.go` rows. Forced by C11.
- C23: `interruptpolicy_meta_test.go` recipe list. Forced by C2 and C5.

No change traces to no decision, and no decision lacks a change that builds it.

**Findings**

None.

**Tool calls**

None made; the inherited context held everything this lens needed.
