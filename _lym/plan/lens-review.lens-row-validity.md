```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked each planned test row: does it reach its branch, would it fail if the code were wrong, and is it needed.

**Findings**

1. **MEDIUM — Card 1, the new `TestGoto_BlockedRunAdmitsOnlyEarlierRows` row can't be built as listed.**
   - The row needs a skipped block row whose `SkipTo` is set to drive `gotoAdmitted`'s routed-row branch.
   - That test only drives the fixed list from `tailShed`/`tailProducers` (`internal/shedengine/goto_test.go:389-398`, rows at `:546-550`), and that list has no row with `SkipTo` and no block row.
   - The row therefore needs a `producers` field on the table, or a change to `internal/shedengine#tailProducers`, which other goto tests share. The card lists neither.
   - Without one of them the row can't put "a skipped block row" in place, so it can't fail if `gotoAdmitted` still reads `OnDone`.

2. **MEDIUM — Card 1, most of the new top-level `TestStep_Block` repeats existing tests.**
   - shedengine has no block concept; the only new engine behaviour is `SkipTo` routing and `Block` stamping. These rows re-test existing behaviour on a renamed list:
     - Holding gate → Done → `OnDone`: `TestStep_EntryGate`'s resume row, and `TestStep_DoneWithNonEmptyOnDone`.
     - The block ending at its exit when the last row returns Done or is skipped: covered by `TestStep_EntryGate`'s skip rows (`entry_test.go:52-117`).
     - Stuck with no `OnStuck` halts, and resume calls the row again: `TestStep_StuckWithNoOnStuck`, `TestRun_ResumeAfterHalt`.
     - Goto onto a gated row skips its gate: `TestGoto_StepResumesAtTarget` (`goto_test.go:233-237`).
     - Relaunch clears the entered row so the gate is evaluated: `TestRelaunch` row "clears the entered row" (`goto_test.go:604`).
   - Fix: keep only the `SkipTo`/`Block` assertions, as rows of `TestStep_EntryGate` (skip by `SkipTo`, inner pass-over to the exit, `Block` on the step, skip and reroute entries).
   - The D3/D4 acceptance criteria are better proven once, over the real expanded arras build in loomrecipe, where the block's meaning lives.

3. **MEDIUM — Card 8, one `TestPrepareRunSettings` row can't fail if `ProfileRecipe` is missing.**
   - The row: "an arras task with no profile named takes tapestry's default profile's tapestry block".
   - The test's shared fixture is `defaults: {darn: light}` (`internal/battencli/runsettings_test.go:21-23`). It has no tapestry default, so `Default("tapestry")` and `Default("arras")` both return "".
   - Over that fixture the row passes whether or not the recipe name is resolved. The card names no fixture that has a tapestry default. The row needs one, or the no-file template, whose default is `tapestry: solid`.

4. **LOW — Card 2, `TestBlockEntry`'s producer half repeats an existing table.**
   - `TestSimpleEntries_HappyPath` is a table keyed by `registryKey` that already covers the no-config `Stub` entry (`internal/shedrecipe/entries_simple_test.go:136,191`). `blockEntry`'s Done-at-once producer belongs as a row there.
   - The unknown-gate refusal is asserted twice: in `TestBlockEntry` and again by card 5's `TestBuild_Errors` row. That refusal is reached through Build, so one assertion is enough.

5. **LOW — Card 9, one `TestRenderChecklist` row repeats an existing case.**
   - "a passed-over `Motif` is boxed skipped as any skipped row" repeats the existing skipped-row case (`internal/loomcli/checklist_test.go:319`). Nothing block-specific runs on that path.

6. **LOW — Card 1, `TestStep_Block`'s gate fake is not on the card.**
   - The Intent drives the block rows with `fakeGate`, but `internal/shedengine#fakeGate` is not under **Uses:**.

7. **LOW — Card 4, the sub-recipe top-level-key refusal is one row for four keys.**
   - "One refusal row per bullet" gives a single row for the bullet covering `contract`, `verify_source`, `amendment_target` and `profile_recipe`.
   - The acceptance criterion names each key, so three of the four go untested. The row should be one table entry per key.

Further tool calls:
- `grep` for the test functions the cards name: `TestShed_Validate`, `TestPrepareRunSettings` and others were not in my glyph listing.
- `sed`/`grep` of `internal/battencli/runsettings_test.go` and `goto_test.go`: needed the `TestPrepareRunSettings` fixture rows and whether any goto test covers the routed-row branch.
- `grep -A45` of `TestGoto_BlockedRunAdmitsOnlyEarlierRows` and `sed` of `TestStatus_JSONRoundTrip`: needed their table shape and helper.
- `grep` for `checklistVisitsOf` callers, the simple-entry `registryKey` table and `tailShed`: needed the re-sign's callers, the table `Block` would join, and the helper's fixed list.
