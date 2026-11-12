```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Behaviour → pinning test**

- Asking rule (no ask yet, another seed, history Discussion-Write after the latest ask, done with findings from the round before, not-sent from an earlier round, satisfied/timeout/no-parent ask nothing): `TestBurlerReviewProducer_Call_ParentAskingRule`, rows at `internal/shedadapters/parentseat_test.go:176-218`. Pinned.
- Key off still posts a `pending-post` ask to its stored strand: the `pending-post` row at `:213`. Pinned.
- An `open` ask is neither posted again nor given a new deadline: the row at `:218`. Pinned.
- No ask sent for a round that sends none, and no seat means no ask: `TestBurlerReviewProducer_Call_WithoutParentSeatAsksNothing`. Pinned.
- Ask that does not go out (no recorded strand, resolve error, post error, chair timeout leaves no room, `not-sent` with reason): `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut`, rows at `:302-314`. Pinned.
- Ask line contents: `TestParentAskLine`. Pinned.
- Settle on acceptance (copy a `done` report, set `open` to `timeout`, both on a spawned review and on an already-accepted review): `TestBurlerReviewProducer_Call_SettlesParentAskOnAcceptance`, rows at `:383-395`. Pinned.
- Ask record functions (`RoundAsk`, `EndParentAsk`, `SettleParentAsk`, the copy-once rule, `ParentCopyPath`, `ReadParentReport`, answer refusals): `parentreview_test.go`, `TestParentAskRecord_RoundAsk`, `TestEndParentAsk`, `TestSettleParentAsk`, `TestParentCopyPath`, `TestReadParentReport`, `TestAnswerParentAsk`. Pinned.
- Parent coverage gate: closes an `open` ask as `timeout`; a satisfied report passes; an uncovered id fails naming it; a merged finding covers both ids; a non-parent origin does not cover; an unparseable review returns findings, not an error: `TestParentCoverageGateEntry_Gate`, rows at `:593-617`. Pinned.
- Gate entry shape (name, no pass on cap, attempts above zero): `TestParentCoverageGateEntry_Shape`. Pinned.
- Gate placed after the review-parse entry, and the parent consolidation part filled whenever the round holds an ask: `TestEngine_Review_TableShape` and `TestParentRulesBlock`. These two tests only show up in the redundancy report; I did not open them. The fill-when-`not-sent` case also depends on `shedadapters` passing the number through for `not-sent`, which `ParentAskThatDoesNotGoOut` does not check.
- `Finding.ParentIDs` parsing: `verdict_test.go:489-494`. Pinned.
- `concludeReview` falls back to `gate.Findings` when `gate.Reason` is empty: `review_test.go:243`. Pinned.
- `PostAsProgram` posts once and returns the line with the cause on a post error: `TestPostAsProgram_PostsOnceThroughTheProgramClient`. Pinned.
- Setting lookup (boolean, unreadable, not a boolean) and the latest `Discussion-Write` stamp (`At` against `LastAt`, other producers, none, unparseable): `TestAskParentSetting_ReadsTheBooleanSetting` and `TestLatestCallAt_ReadsTheLatestProducerStamp`. Pinned.
- Parent seat lands on the Discussion segment only: `TestLoomSegmentSettings_ParentSeatLandsOnTheDiscussionSegmentOnly`. Pinned.
- Review keys reach the env: `TestWire_ReviewKeysReachTheEnv`. Pinned.

**Findings (no pinning test)**

1. **MEDIUM, design.** `internal/shedadapters/parentseat.go:49-55` (`roundSeed`, compared in `RoundAsk` and `parentAskDue`). D2 requires the seed at full nanosecond precision, compared exactly. The fixture seed is `parentSeedTime`, a whole-second time (`parentseat_test.go:37`), and the "another seed" rows use a different string (`otherParentSeed`, `:199-204`). No row has two seeds that differ only in nanoseconds, so truncating the seed to seconds would break nothing. Fix: add a row to the asking-rule test, or a `roundSeed` case, with two seeds one nanosecond apart that must ask again.
2. **LOW, design.** `parentseat.go:109-115` and `:118-122`. The failure branches `RecordWrittenAt` error ("judged as though not rewritten") and `ReadParentReport` error ("asks again") have no row. My grep of the test file found `RecordWrittenAt` only in the fixture closure at `:88`, which always returns nil, and no unreadable-report row. Fix: add rows for a `RecordWrittenAt` error and for a missing report file.
3. **LOW, design.** `parentseat.go:59-73` (`readParentAsks`). An unreadable seed (no focus file) or an unreadable ask record, which should go on without the parent, has no row. Fix: add a row with no round-1 focus file and a row with a corrupt `asks.json`, each expecting no ask and no post.
4. **LOW, design.** `parentseat.go:168-180` (`postParentAsk`). I found no test that the `no-parent` set on a post error and the `MarkParentAskPosted` failure paths log and carry on. The post-error row at `:310` checks the state only through the main flow, so I cannot confirm the `EndParentAsk` failure branch is reached. Fix: none needed unless the state assertion is missing; confirm the row asserts the final `no-parent` state.
5. **LOW, design.** `internal/burlerengine/parentseat.go:79-92`. The gate's error and failing branches for a `done` ask have no row: an unreadable report file (returns an error) and a missing review file (returns findings). Only the unparseable-review case is covered. Fix: add two rows to `TestParentCoverageGateEntry_Gate`.
6. **LOW, design.** `internal/orchcli/notify.go:382-384` (`ParentStrand`). It forwards to `parentOrchStrand` with `OrchLive`, and I found no test of the wrapper. The routing is probably covered through the existing `NotifyParent` tests. Fix: nothing, unless the wrapper's live-orch argument matters; then add one row.
7. **LOW, design.** `PostAsProgram`'s claim that it ignores a non-orch strand claim holds by construction: `NewScopedProgramClient(location, "", program)` is untested. `TestPostAsProgram_PostsOnceThroughTheProgramClient` uses a fake poster, so it cannot fail if the real call passes the process's strand name in place of `""`. D3's acceptance criterion "a review row whose process carries a non-orch strand name posts signed as `&lt;shortname&gt;:LOOM`" is therefore not pinned. Fix: add a test with a non-orch strand-name env that checks the sender through the real scoped client, as the existing `NotifyPrime` tests do for claims.

**Tool calls (3)**
1. grep of the test files' function and row names, because my context held only the production diff.
2. Read of `parentreview_test.go:565-660` plus a grep of `verdict_test.go` and `review_test.go`, to see the gate rows, the `ParentIDs` test and the `gate.Findings` fallback test.
3. grep of `parentseat_test.go` for seed, `RecordWrittenAt` and error rows, to settle findings 1-3.
