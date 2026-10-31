```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the plan (cards 1–20): every decision, criterion and branch traced to a card and a test row.

**Findings**

1. MEDIUM — D4, criterion "an uncommitted pair … commits" (card 11). `TestRecordWrite_Call` has no row with an existing pair that is still uncommitted. Its "existing pair is left byte for byte" row asserts nothing about the commit. So if Call only committed after writing a file, it would pass the planned rows but strand a pair a crash left uncommitted. Evidence: `plan:internal/loomshed#TestRecordWrite_Call` rows in card 11.

2. MEDIUM — D4, criterion "that route reaches Darn-Write both on a task branch with no commit past the branch point and on one with commits whose scope record holds `awaited_at`" (card 15). Card 15 only argues this from `internal/driftengine/doc.go`; no row walks goto Record-Write → Drift-Baseline → Drift-Await → Darn-Write. Evidence: card 15's "That route covers a run in flight…" paragraph, with no matching test.

3. MEDIUM — D6, criteria not reached by any row (cards 13, 15):
   - an answer recorded with `lym loom decision add` reaches the resumed writer and the Code-Review perch as an `Added after Discussion` entry;
   - a no-parent run whose board holds an answering `Decision for <slug>:` note, but whose record holds no such entry, halts again on resume.
   Both are argued from the stencil alone. The second one can be reached mechanically: a resume whose fake session writes the stop marker again halts again. Evidence: card 15's test list.

4. MEDIUM — D6, fidelity to the record (card 15). D6 says the writer writes the stop marker "in the run's directory". Card 15 puts `shedrun.StopMarker` in the run's scratch directory, which is ephemeral `.lym`, not `shedrun#RunDir`. Evidence: card 15, "`shedrun.StopMarker` names the marker in the run's scratch directory (never tracked)".

5. LOW — D6 (card 15). The halt reason is "the change description's text read from the run's description", but `DarnDeps` gains no description path or reader. Only `RecordPath`, `CommittedRecord` and `StopMarkerPath` are added. The card should name its source, for example the session pointer's Path. No row pins that the reason equals the description's bytes. Evidence: card 15's `DarnDeps` bullet.

6. MEDIUM — D10, criterion "a broken batcher.yaml fails Preflight on a tapestry run and not on a darn run" (card 9). `TestPreflight_BatcherCheck` drives only the producer seam. No row asserts that `wire` sets `PreflightBatcher` for tapestry and leaves it nil for darn and pawl (`loomrecipe.HasEngine(c.recipe, "Implementer")`). `TestWire_DarnRecipe` and `TestWire_DefaultConfig` are not edited by card 9, and `nilLegal` would hide a nil on tapestry. Evidence: card 9's Edit and Tests lists.

7. MEDIUM — card 1, branch "a backtick run whose info string holds a backtick opens none". No row reaches it. A closing run of the other character (tilde vs backtick) is not reached either. Evidence: card 1's rows for `TestAppendDecision_AppendsAtEndOfDecisionsSection` and `TestValidate_MissingHeadings`.

8. MEDIUM — card 3, clause 1/2 branch "two build rows". The test has a row for two record rows, but none for two build rows (two of `Implementer`/`DarnWrite`). Evidence: `plan:internal/shedbuild#TestBuild_LoomContract` rows in card 3.

9. MEDIUM — card 8, new refusal. `New(recipe, …)` refusing a recipe that does not declare `contract: loom` (naming `NewPawl`/`lym shed`) has no reaching row. The card plans no new test and only converts existing callers. This breaks PATTERN-refusal-way-forward. Evidence: card 8's Intent, first bullet.

10. MEDIUM — D3 / card 20, `peerRecipe`'s fallback branches. The fake `rows` func in `peersFakeDeps` bypasses `peerRecipe` entirely. `TestHubPeersDeps_OverRealHub` adds only a seed-records-darn pair. These branches are unreached: off-machine project entry → pawl, off-machine entry `recipe: darn`, empty recipe field → tapestry, and a seed recording an empty recipe → tapestry. The D3 criterion "an off-machine pawl peer's row, from a project entry with an empty `recipe` field" is then pinned only through the fake, not through `peerRecipe`. Evidence: card 20's Tests paragraph.

11. MEDIUM — card 17, `wire`'s new stat branch. It passes `RunRecordPath` only when the webster summary exists, and a stat error is returned. No row reaches either side, so a darn run ending up with a non-existent run record path is unpinned. Evidence: card 17, the third bullet and its test list.

12. MEDIUM — card 18, behavior change. A darn run's review rows are now told `PatternRecordPath` (D3, so the Code-Review perch checks declared PATTERN changes). `TestRecipeKeyedWiring` does not assert it, and no other listed row does. Evidence: card 18's `wire` bullet and its test paragraph.

13. LOW — card 18, unreached branches. `landingDeps`' `EscalateJudgedFirst` via `IsLoom`, and the failure tracer keyed on a `PlanWrite` row, have no row. Evidence: card 18's `TestRecipeKeyedWiring` paragraph.

14. LOW — card 13, new refusal. `resolvePlainGateClosure`'s `"discussion"` now refuses a nil `Env.DiscussionHistory` (`requireSeam`), but no row reaches it. `TestResolveGateSpec_ValidOneEntryPerVocabularyName` only builds both forms. Evidence: card 13's resolver bullet and its tests.

15. LOW — D13, "refused at its write and at Seed-Child". Only `TestPrepareRunSettings` reaches the refusal. Nothing pins that it carries the same Spec text as the local-file refusal (the card asserts "naming the key and shed", not equality). Pawl taking `shed.yaml`'s value is not in `TestTacklerFor`/`TestHoldReading`, which list only darn and tapestry. Evidence: card 5's Tests.

16. LOW — card 6, fallback "empty `plan_judge`/`plan_seed` takes `loom.judge`/`seed`". `TestLoadTapestryConfig` and `TestResolveReview` have no row for it. Evidence: card 6's Tests paragraph.

17. LOW — card 9, branch "a cancelled context wins" in `NewPreflightWithBatcher`'s batcher check. `TestPreflight_BatcherCheck` has no row for it. Evidence: card 9's Preflight bullet.

18. Other (outside my lens), LOW–MEDIUM — card 6. `LoadConfig` of loom now calls `LoadTapestryConfig`, which loads `tapestry.yaml` strictly. So every loom, darn and pawl verb that loads `loom.yaml` refuses until a reconcile creates a `tapestry.yaml` base. No row covers a hub not yet reconciled, and the card does not say how the strict load stays satisfied. Evidence: card 6, "`LoadConfig` fills them from `LoadTapestryConfig` after its own checks"; `internal/loomengine/config.go` `LoadConfig` uses `configengine.Load` (strict).

No further tool calls were made; my context held everything this lens needed.
