```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the plan's test rows. 3 MEDIUM, 6 LOW.

**MEDIUM — Card 12, the test wiring and the claim that entry configs can't be read back.**
The premise is false: a precedent test already checks an Env setting reaches the built Bouncer, `internal/shedrecipe/entries_bouncer_test.go:526` (`t.Run("EnvReviewCirclingCheckpointReachesTheBouncer"…)`).
- No row checks that `Env.ReviewStalemateRounds` reaches `BouncerConfig.StalemateRounds`.
- No row checks that `BouncerConfig.Snapshot` and `BurlerHalfDeps.Snapshot` are set.
- A dropped wire leaves the zero value (`nil-seams-keep-today`), which turns the stalemate check and incremental rounds off silently, and no planned row fails.
- Fix: add a "reaches the Bouncer" row to that test for `ReviewStalemateRounds` and a non-nil Snapshot. List `bouncerEntry`'s test glyph on Card 12. Drop the sentence that says the configs cannot be read back.

**MEDIUM — Card 4, the "second Call keeps the first target id" row can't fail.**
- After a first `Call` whose review is accepted, the acceptance record exists. A second `Call` therefore hands on at `BurlerReviewProducer.Call` (`internal/shedadapters/burlerreview.go:264-266`) and never reaches `recordRoundStart`.
- The row passes whether or not `recordRoundStart` keeps an existing record.
- To reach the keep-first branch, the first attempt must end without an accepted review. For example, a `ReviewErrs` entry fails the first `Call` through `failureExit`, which archives only the paths list, not the snapshot record. The second `Call` then re-enters `recordRoundStart`.

**MEDIUM — Cards 4 and 7, a test helper the rows need is missing from both cards.**
- `newReviewProducer` (`internal/shedadapters/burlerreview_test.go:55-57`) builds `BurlerHalfDeps` from `halfProducerSpec` fields. That type has no `Snapshot` or `DecisionRecordPath` field.
- The rows in Card 4 (`TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`) and Card 7 (`TestBurlerReviewProducer_Call_ProfileAndOpts`) need both.
- `newReviewProducer` sits under **Uses** on both cards, and `halfProducerSpec` is listed nowhere. Card 4 must list `internal/shedadapters#newReviewProducer` and `#halfProducerSpec` under **Edit**.
- Also on Card 4: `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded` is a two-row table over `{name, engine}` that delegates to `requireAcceptedReviewRecorded`, a usage-record helper (`burlerreview_test.go:529-569`). Snapshot rows don't fit its shape. They need their own fields or helper, and the card names neither.

**LOW — Card 7, the `ProfileAndOpts` changes-file row lacks its setup.**
- Round 1 must read as judged, or `Call` hands on (`burlerreview.go:251-255`). The helper for that is `writeJudgedRound` (`internal/shedadapters/burler_test.go`), which the card doesn't list.
- An overlay changes file also needs round 1's cache copy at `CacheDir/<id>`, so the row must call `Take` or drive round 1 through the producer. The card names neither.

**LOW — Card 9, the empty-slug branch of `noticeWayForward` can't be reached.**
- `renderEscalation` fills the notice stencil with `stencil.Fill`, which errors on an empty marker (`internal/shedadapters/bouncer.go:1577-1582`, and the comment at `bouncer.go:1826`).
- So an empty `Slug` gives no notice at all, and the "drop the slug argument" text is never sent.
- `TestBouncer_Circling_NoDecisionAwaitsNamingBothVerbs` runs rows over `tt.slug` through `circlingBouncer(t, func(c){c.Slug = tt.slug})` (`internal/shedadapters/bouncer_circling_test.go:110`). The new `ParentNotice` assertion fails on any empty-slug row unless it is limited to the slugged rows. Drop the empty-slug clause or say the notice is absent then.

**LOW — Card 4, the "a skipped fix records it too" row repeats another row.**
`decisions_routed` is written before the runner is called, and the fake runner's `Skipped` result never reaches a distinct path (`internal/shedadapters/burlerfix_test.go:189`). The row repeats the notifier row.

**LOW — Card 2, one assertion is planned twice.**
"A Go-written skipped report parses with `Spawned` false" is planned in both `TestCheckFixerReport` and `TestEngine_Fix_Outcomes`. Both run in package `burlerengine` and can read `skippedFixerReport`. Keep one.

**LOW — Card 8, a tier1 test lands in a package whose test file is tier0.**
- `TestValidateReviewVerb_RoundChecks` is tier1, but `internal/burlercli/cli_test.go` is all `tiergate.Require(t, 0)` (`cli_test.go:31,52,261`).
- PATTERN-test-speed allows one tier per file, so the test needs a new file. The card declares it only as a package handle and names no **Create** path with its tier.

**LOW — Card 5 names a glyph its body never uses.**
Card 5's `**Uses:** internal/burlerengine#FixerReport` is referenced nowhere in its body. `RoundContext` uses `*FindingDisposition` alone.

Tool calls I made:
1. Grepped `review_test.go` test names and resolved Card 5/7/8 glyphs. My context lacked the review-half test names and those symbols' existence.
2. Read `fakeHandle.evaluateGate` and `halfScript` in `engine_test.go`. I needed whether the fake honours `PassOnCap` and re-prompt rewrites.
3. Read `newHalfSpec`, `newReviewProducer` and `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`. I needed the helper's deps fields and the test's table shape.
4. Read `TestBurlerFixProducer_Call_FinishedFixIsRecorded`, `newFixProducer` and the Bouncer decision and circling helpers. I needed their fields and tweak hooks.
5. Grepped the circling test's slug rows and `withEscalationInputs`. I needed whether empty-slug rows exist under the new notice assertion.
6. Grepped `entries_bouncer_test.go` for how Env settings are asserted to reach the Bouncer. I needed to test Card 12's claim that this can't be tested.
