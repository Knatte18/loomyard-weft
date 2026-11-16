```yaml
lens: row-validity
outcome: reported
```

Review of the 26-card plan through the row-validity lens: can each planned test row reach the branch its card claims, would it fail if the code were wrong, and is it needed?

**Findings**

1. **BLOCKING** — Card 21 (D13 decision-notice guard): the delivery cannot be recorded as specified, and the row claiming it cannot pass.
   - `notifyLiveRuns` only holds strand *names* from `deps.liveStrands` (`internal/loomcli/decision_test.go:88`, `:299`; fake values look like `"hub:task-a:webster"`).
   - `RecordDelivery(anchorRoot, strandGUID, headings)` keys on a strand guid, and card 5's `AttemptState` keys on `StrandGUID`. Nothing on the notice path turns a name into a guid.
   - `decisionDeps` carries no anchor and no delivery seam (`internal/loomcli/decision.go:263-303`), and the card lists no new `decisionDeps` field and no fake for it.
   - So the row "a running Weaver run's batch strand gets the inline notice and the delivery is recorded" cannot reach a recorded delivery, and its assertion cannot be written against the existing fake.
   - Fix: key the delivery by strand name (or record the name in `AttemptState`/`FixerState`), add a `recordDelivery` field to `decisionDeps`, and list that field and its fake on the card.

2. **MEDIUM** — Card 9 `classifyAttempt` (D6, D12): one listed verdict cannot reach the function.
   - The signature takes `shuttleengine.Result, waitErr, reportPath`.
   - "A respawn-eligible attach clear" is the attach's not-found answer: `AttachGated` returns `(Result{}, false, nil)` (`internal/shuttleengine/attach.go:59`). No `Result` value carries it.
   - `TestClassifyAttempt`'s "one row per verdict" therefore cannot hold a row for it. That not-found branch belongs to card 13's `resumeStrands`, and `TestRun_Resume` lists no row for "attach answers not found, no report on disk → machinery loss".
   - Fix: drop that verdict from `classifyAttempt`, and add the not-found-without-report row to `TestRun_Resume`.

3. **MEDIUM** — Card 4 (test economy): `TestDoneCards` in runplan repeats tests that already exist.
   - `TestLastBegunCard` (`internal/websterengine/begun_test.go:50`) and `TestEditedDoneCards` (`:84`) already cover these bodies.
   - Card 4 moves the bodies to runplan and keeps the websterengine functions as one-line delegates. Both suites would then cover the same behaviour.
   - Fix: move those tests into runplan, re-pointed at the runplan functions, or keep them where they are and drop the new runplan test. Do not have both.

4. **MEDIUM** — Card 20 (D13 renumber): the rows never reach the seam or the lease choice.
   - `TestRenumberVerbs_Refusals` fakes `renumberDeps.loadState` and `acquireLease` directly (see the subagent catalog of `renumber_test.go`).
   - A "Weaver state's begun bound" row just feeds a `*runplan.State`, which the existing done-batch row already does.
   - "Both implementers begun refuses with the seam's text" is a faked error, the same branch as the existing "run record that fails to load" row (`errRenumberRawIO`).
   - The new behaviour lives in `newRenumberDeps`: `weaverengine.Active`, and the lease taken in `RunDirs`' scratch directory. No row covers it, so the acceptance criterion "a renumber racing a Weaver batch begin is serialized by the state-mutation lease" has no test that would fail.
   - Fix: add a row driving `newRenumberDeps` (or an extracted builder) over planted Weaver state, asserting the bound and that the lease path sits under `.lym/weaver/`. Drop the two duplicate rows.

5. **MEDIUM** — Card 25 (D10 batten timeout): the planned rows would not fail if Weaver's key were ignored.
   - The existing steps register processes at `time.Now()` or `-24h` (`internal/battencli/wire_seams_test.go:612-623`).
   - With Webster's `recovery_timeout_min` and Weaver's `fixer_timeout_min` both defaulting to 60 (`websterengine/template.yaml:5`, card 2's template), "within → wait" and "past → no wait" pass whichever config `fixerTimeout` reads.
   - Fix: the card must plant Weaver state, or set `tapestry.implementer: weaver`, in the task location so the seam answers Weaver. It must also write a `weaver.yaml` whose `fixer_timeout_min` differs from Webster's, and add a step registered between the two bounds. The card lists no such fixture.

6. **MEDIUM** — Card 22 (D13 Describe and rework archive): the two wiring changes have no row.
   - `DescribeTable` now reads `RunRecordPath` through `weaverengine.Active`, and the `ArchiveWebster` closure now also archives into the `weaver` slot.
   - No loomcli test exercises either. `TestWire_DescribeTableCarriesTheDriftVerdict` checks only the verdict, and nothing calls the `ArchiveWebster` closure (subagent catalog, items 3 and 7).
   - So the criteria "Describe reads Weaver's summary.md" and "a PR rework round archives `_lym/weaver/` into `prior-generation/weaver`" are untested; only `ArchivedRunRecordDirs` is tested.
   - Fix: add rows that drive the seams over a planted Weaver run directory.

7. **MEDIUM** — Card 24 (D8 arbiter note): no row reaches `arbiterDepsFor`, where the active-implementer read happens.
   - `TestRecipeKeyedWiring` calls `arbiterRecipeDescription(tt.recipe)` directly (`internal/loomcli/recipekeyed_test.go:70`).
   - Once the planned rows hand it a note, they test only string concatenation. The seam read in `arbiterDepsFor` and the `ArbiterNote` rendering for Webster versus Weaver have no row. `ArbiterNote` is a new exported function with no test of its own.
   - Fix: add an `ArbiterNote` table, or put the note rows on a test of `arbiterDepsFor`.

8. **LOW** — Card 17 (D1 guard): the "after each named reset" rows do not run the reset.
   - `TestImplementerGuard` plants the archived-state shape by hand. That repeats the row "neither begun proceeds".
   - The criterion "a test runs each named step and the row then proceeds" is not met; the reset verbs never run before the guard.
   - Fix: drop the duplicate rows, or drive `websterengine.ArchiveRunAfterReset` and `weaverengine.Reset` (card 14) before the guard.

9. **LOW** — Card 19 `TestNewPlanGate` (D13 plan gate): the new rows need a config path the card does not state.
   - `ValidatePlan` reads the `tapestry.implementer` setting "from the anchor's config dirs". In a temp-anchor tier0 test that load must degrade to the template.
   - `loomengine`'s tapestry load is strict (`internal/loomengine/tapestry.go:207`, through `loadResolved`), and it errors "not initialized" on an absent hub config.
   - So the rows are unreachable, or error, unless the setting is told as a seam or the read degrades. The card names no such seam or fixture.
   - Fix: state how `ValidatePlan` gets the setting, for example a told lookup, and give the test a fake for it.

10. **LOW** — Card 2 (config): the claim that `TestModules_EveryTemplateKeyClassified` needs no new row is unconfirmed.
    - For a per-worktree module, the test compares `m.LocalDenied` to `wantLocalDenied[m.Name]` (`internal/configreg/configreg_test.go:143`, `:176`).
    - With every key `CategoryNone`, `LocalDenied` should come out empty and match a nil entry. The card should say so, or the test may want an explicit `weaver` entry.

Further tool calls (1):
- One grep/sed over `websterengine/begun_test.go`, `loomcli/recipekeyed_test.go`, `loomcli/decision_test.go`, `battencli/wire_seams_test.go` and `configreg/configreg_test.go`. My context had their names but not the existing tests' bodies and fixtures, which this lens has to open.
