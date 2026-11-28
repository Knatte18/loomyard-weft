```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the change set agrees with its sibling cards, decisions and cross-references (fix-consistency lens).

**LOW — card 12, "Support for the moved tests" (`newReplayHub` / `pushPrimeRecords` bullet): the moved reconcile rows have no named hub builder.**
The old text said the reconcile rows build on `reconcileHub`/`newReconcileHub`, "a hubforge pair with the replay installed and the prime's records published".
The rewrite drops those Create lines and describes `newReplayHub` and `pushPrimeRecords` only in push-row terms, such as "pushes the prime's records branch with its upstream set".
It never says that `TestReconcileOrigin_EndedRuns` and `TestReconcileOrigin_OwnedElsewhere` build their hub from `newReplayHub` plus `pushPrimeRecords`.
It also never says how those rows get the pair that the old fixture added.
A reader can't tell what replaces the reconcile fixture.

**LOW — card 12, ImpactSummary: "no production code changes" now sits next to a non-test file edit.**
The same card now adds `internal/testkit/boardkit#BoardFile` and edits `internal/testkit/boardkit#WriteBoard` in a non-test file.
The summary does say "boardkit gains its board-file encoder", so the claim is only loosely wrong: a reader may count the kit file as production.

Siblings checked with no disagreement found:
- `00-overview.md`'s reworded sequential-seam decision agrees with card 4 (line 68) and card 9 (line 93), which each build a hub first, and with card 7's two no-hub in-package tests.
- No card other than 12 names a helper card 12 removed or deleted: `reconcileHub`, `failureFixture`, `pairFixture`, `pushPair`, `fakePushNotifier`, `statusJSON`, `boardFileOf`, `countingBackoff`, `requireOneFailedPush` or `requireNoFailedPush`.
- In card 7, `OutcomeProtectedUnrejected` appears consistently in the Create line, the `outcomeClassOf` mapping, the `TestOutcomeClassOf` row, the loop rule, the verdict rule, the `protectedParentWayForward` rule and the `TestPushSide` row. The `FieldPushJudgment` read condition at line 79 still holds.
- Card 3's rewritten search sentence matches card 7's new Edit lines for `pushdeferral_integration_test.go`.

Tool calls:
- One grep over cards 00–11 for references to the removed helper names, the protected outcome classes and the hub-first wording, to find sibling twins; my context held no other card's text for those names.
