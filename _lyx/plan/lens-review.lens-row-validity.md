```yaml
lens: row-validity
outcome: reported
```

Row-validity review: I checked whether each planned test row reaches its branch and would fail if the code were wrong. I found one BLOCKING, three MEDIUM and two LOW defects.

1. **BLOCKING — Card 4 (D2, `lyx loom review` is no command).** An existing test still looks up the removed `review` group, and no card lists it.
   - `internal/loomcli/cli_test.go:30-48`: `TestResolvePersistentPreRun_SkipsReviewGroup` calls `parent.Find([]string{"review"})` and fails fatally unless it finds a command named `review`.
   - Once the `review` subtree is gone, card 4's own gate fails.
   - Fix: list `internal/loomcli#TestResolvePersistentPreRun_SkipsReviewGroup` under card 4's `**Edit:**` and point it at the `circling`, `decision` and `peers` groups.
   - The new `TestCommand_HasNoReviewGroup` should then be an assertion in that rewritten test, not a new top-level test (Test economy).

2. **MEDIUM — Card 13 (D8 Bouncer switch).** The planned row on `TestBouncerEntry_BlockingEscalatesAndColorSegment` cannot fail if `escalate_decisions` is ignored.
   - `internal/shedrecipe/entries_bouncer_test.go:1243-1290`: the test asserts only behaviour (`wantOutcome`), since the Bouncer's `cfg` is unexported.
   - `internal/shedrecipe/entries_bouncer_test.go:570`: its fixture `layoutBouncerRound1Report` writes a review with `findings: []`.
   - With no `decision`-class finding in the review, the row behaves the same whether the key reaches `EscalateDecisions` or not.
   - Fix: the card names a round-1 review holding a `decision` finding (an override or a helper, listed on the card), and the row wants `Awaiting` with cause `decision`.

3. **MEDIUM — Card 12, row 3 (`discussion_auto_accept` true at the soft cap escalates `decision` instead of accepting).** The row cannot reach the auto-accept branch.
   - `internal/shedadapters/bouncer.go:1274`: auto-accept fires only when `spawned` is true, meaning the judge was harvested in this Call.
   - `decisionBouncer`, as declared on the card, lays out the round's verdict and ledger on disk through `layoutBouncerRun`, so `Call` takes the replay path at `internal/shedadapters/bouncer.go:493-497` with `spawned` false.
   - On a replay, auto-accept never fires whatever the routing does, so the row passes even with the routing removed.
   - Fix: round 2 starts unjudged and the fake shuttle writes the verdict and ledger during the judge spawn, as `judgeFakeShuttle` does.

4. **MEDIUM — Card 13 (fix-row switch).** The row added to `TestBurlerFixEntry_RoutesDecisionFindingsThroughEnvNotifyParent` is stated as "asserts `decisions_escalated: true` reaches the deps", which the test cannot observe directly.
   - `internal/shedrecipe/entries_burlerhalves_test.go:746-772`: the test observes only the posted `lines` and `fake.GotFixOpts[0].DecisionsRouted`.
   - Fix: the row sets both `env.NotifyParent` and `decisions_escalated: true`, and wants zero posted lines with `DecisionsRouted` true.
   - Otherwise an implementation that leaves the notice in place passes.

5. **LOW — Card 12, row 13 (re-entry after a settled `decision` accept).** The row needs a `Subject` seam, settle and generation records, and a live subject it can change. The card's `**Uses:**` lists no such fixture: neither the existing `internal/shedadapters#newReentry` and `internal/shedadapters#convergingShuttle` helpers (`bouncer_reentry_test.go:49-102`) nor a fake `Subject` on `decisionBouncer`. Row 4's "changed `Subject` between the calls" needs the same seam.

6. **LOW — Card 6 (`TestPrepareRunSettings`).** The replacement row's expected output is unstated.
   - `internal/configreg/runsettings_test.go:223-232`: the existing rows assert `wantRefusal` or `wantDocs` exactly.
   - Once the migration drops `skip_alignment: true`, the loom object is empty, and the card does not say whether the row wants no `loom` document (a module whose object is empty is dropped) or an empty one.
   - The criterion ("loads with a Warn") is not asserted either: no log capture is named.

Per row:
- **Card 3:** `TestResolveGateSpec_UnknownNameListsTheLegalValues` reaches its branch and fails if wrong; it replaces an existing test, so it is no repeat. `TestDiscussionWriteRow_GateListIsDiscussionAlone` reaches and fails if wrong; it is new.
- **Card 4:** `TestCommand_HasNoReviewGroup` is a repeat (finding 1). The fake-next rewrites of `TestShuttleWaiting` and `TestVerifyWaiting` reach their branches.
- **Card 7:** the `TestConfigTemplate_DecodesAndIsSelfConsistent` rows reach and fail if wrong. The `TestLoad` row is new as far as I read, though I did not read all of `TestLoad`.
- **Card 11:** reaches and fails if wrong.
- **Card 12:** rows 1, 2, 5 to 12 and 14 reach via the replay path and fail if wrong. Row 3 does not reach (finding 3); rows 4 and 13 depend on an unlisted fixture (finding 5).
- **Card 13:** the recipe test reaches and is new; the two entry-test rows are findings 2 and 4.
- **Card 14:** the new `decision` row reaches and fails if wrong; running `Check` in the refused rows is feasible over a `Readers` fake.
- **Cards 15 and 16:** the migration rows reach their branches; `TestRenameRetiredRows` is new, since no existing test covers `renameRetiredDarnRows`.

Tool calls beyond my context:
- `grep` over `cli_test.go`, the rename tests and the carry-over tests: to find existing tests of the CLI tree and the row rename, which my context lacked.
- `sed` of `cli_test.go:30-52` and a `grep` of the two entry tests: to read how the pre-run skip test and the entry tests assert, which my context lacked.
- `sed` of `TestBouncerEntry_BlockingEscalatesAndColorSegment` and `TestBurlerFixEntry_RoutesDecisionFindingsThroughEnvNotifyParent`: to read their full assertions.
- `sed` and `grep` of `TestEscalationEligible`, the `configreg` alignment rows and the lens override rows: to check the shape of the rows cards 6, 7 and 14 extend.
- `grep` of `TestLoad` and of `layoutBouncerRound1Report`'s review content: to check the `TestLoad` override rows and the round-1 review fixture card 13's row depends on.
