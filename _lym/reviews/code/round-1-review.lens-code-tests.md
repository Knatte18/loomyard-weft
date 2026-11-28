```yaml
lens: code-tests
outcome: reported
```

I covered the code-tests lens: for each behaviour the diff adds or changes, the test that would fail if it broke.

**Behaviour → pinning test**
- Expand admits a last outermost block row with an always gate, and its inlined last row keeps an empty on_done → `expand_test.go:108` row.
- Expand refuses a last empty-exit block row whose gate is not always, with its way forward → `expand_test.go:271` row.
- Expand still refuses an empty-exit block row that is not last → `expand_test.go:265` row, which predates the diff.
- Expand holds a nested block row, whose empty exit comes from a run-ending outer block, to the last-row and always-gate rule (a summary design decision) → none.
- Loom contract refuses neither, two record rows, two seed rows, and a seed row ahead of its record row → `build_test.go` `TestBuild_LoomContract` rows.
- Loom contract reads the seed row as the record row for later clauses → `build_test.go` row "a seed row standing for the record row after the build row".
- Loom contract admits a seed row after, or in place of, the record row → the `build_test.go` `seedForms` rows.
- Loom contract builds tapestry, jacquard, darn and motif → `New` in `TestJacquardRecipeRunsAlone`, `TestRecipeKeys` and the shape and sequence tests.
- Record-Seed seeds a missing record and support log from the stored record and commits → `recordseed_test.go` `MissingRecordIsSeededFromTheStoredRecord`.
- Record-Seed leaves an existing pair byte for byte and still commits → `ExistingRecordIsLeftAsItIsAndStillCommitted`.
- Record-Seed with the record present and the support log missing writes only the log → none in Record-Seed's tests.
  The path is shared with Record-Write, so its tests in `recordwrite_test.go` may cover it.
- Record-Seed is Stuck with nothing written, and its reason gives the Discussion goto or the batten and re-seed sequences → the `NoStoredRecordIsStuck_*` rows.
- A ReadSeed failure is returned → `ReadSeedErrorIsReturned`, which checks `errors.Is` only, never the message.
- Record-Seed honours the cancel check before its Stuck → none.
- The `recordSeedEntry` construction refusals and its build over a filled Env → `entries_recordseed_test.go`.
- `recordSeedDeps` wiring: DiscussionRow per recipe (Discussion-Write for tapestry, empty for jacquard), the seed path, and the driver, parent and recipe read from the real seed → none (seam tests check only that seams are non-nil).
- `RecordEngine` returns RecordSeed for jacquard and DiscussionWrite for tapestry → `recipekeys_test.go` rows `JacquardRunsAlone` and the tapestry case.
- The jacquard top-level keys and the tapestry block expansion → `blockrecipes_test.go` `TestTapestryBlocks`.
- Jacquard builds and runs alone → `TestJacquardRecipeRunsAlone`.
- The undiscussed tapestry walk through Motif, Discussion, Jacquard, Preflight, Record-Seed (writing nothing) and Drift-Baseline → `TestSequence_FullRunBlocksAtMergeIn`.
- A discussed entry passes over Motif and Record-Seed writes nothing → `TestSequence_DiscussedEntryPassesOverMotif`.
- jacquard in the vocabulary and the shedcli table → `seed_test.go` `TestRecipeNames` and `table_test.go`.
- The interrupt policies for Record-Seed and Jacquard → `interruptpolicy_meta_test.go`.
- Stage numbers and the past-approval flip for Preflight → the `progress_test.go`, `recipe_test.go`, `discussionapproval_test.go` and `battencli overview_test.go` rows.
- `BouncerRunSubdir` and `IsFinalizeRow` listing jacquard → none, but nothing observable changes: tapestry, checked first, already holds every jacquard row.

**Findings**

1. MEDIUM, design. `internal/loomcli/wiring.go:552-572` (`recordSeedDeps`).
   No test pins the wiring that fills the Stuck reason.
   That wiring covers DiscussionRow from `EngineRow` per recipe, `ReadSeed` mapping `seed.Driver` and `seed.Params["parent"]`, Recipe set to tapestry, and `SeedPath` from `shedrun.SeedFile`.
   A wrong param key or row lookup would pass every test.
   Evidence: `recordseed_test.go:28-50` replaces these seams with stubs, and a grep of test files finds no `recordSeedDeps` and no DiscussionRow assertion outside `recordseed_test.go`.
2. LOW, design. `internal/loomshed/recordwrite.go:185-187`.
   The new cancel check before the stored-only Stuck has no test.
   The `CancelledContextWins` test covers Record-Write only.
   Evidence: `recordwrite_test.go:311`, and no cancel case in `recordseed_test.go`.
3. LOW, design. `internal/loomshed/recordseed_test.go:331-340`.
   The ReadSeed failure refusal ("read the run's seed", with its way forward) is reached, but its message is not asserted, as `PATTERN-refusal-way-forward` requires.
   Evidence: the test checks `errors.Is(err, boom)` only.
4. LOW, design. `internal/shedbuild/expand.go:93-97`.
   The nested run-ending block case has no test row: a block row inside the last always-gated block, whose exit stays empty, must itself be last and always gated.
   The summary's "Design decisions" claims this behaviour.
   Evidence: the `expand_test.go` row names (lines 75-283) include no nested empty-exit case.

Tool calls:
1. A grep of `recordwrite_test.go` for cancel, of test files for `recordSeedDeps` or DiscussionRow, and of the `expand_test.go` row names. My context had not read these test files.
