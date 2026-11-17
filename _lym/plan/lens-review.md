# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates are merged: the `stalemated` order finding came from way-forward 1, coverage 6 and guard-reach; the entry-wiring test finding came from row-validity M1 and coverage 5.

- forced-edits 1 — BLOCKING — fixed as proposed: Card 12 lists `internal/battencli#battenCLI.wire` and `internal/testkit/envkit#FullEnv` under Edit and fills both new closures there, and lists the loomcli and battencli `TestWire_EverySeamFilled`, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe` and `TestFullEnv_FillsEverySeam` under Uses.
- way-forward 1 / coverage 6 / guard-reach — MEDIUM — fixed as proposed: Card 10's `stalemated` checks round N's own `stalemate` escalation record before the `StalemateRounds == 0` test, and `TestBouncer_Stalemate` gains a replay row at `StalemateRounds` 0 with a CONTINUE verdict below the window, plus a row whose own record holds another cause.
- row-validity M1 / coverage 5 — MEDIUM — fixed differently: row-validity proposed extending `EnvReviewCirclingCheckpointReachesTheBouncer`, but that row only checks construction succeeds (`internal/shedrecipe/entries_bouncer_test.go:526-533`), so a dropped wire would still pass it; Card 12 instead adds a row to `TestBouncerEntry_CirclingReasonSlugAndBudget` that drives the built Bouncer to a `stalemate` Awaiting (and not at 0), and a row to `TestBurlerReviewEntry_ProfileMapping` that calls the built review producer and reads `round-1-snapshot.json`'s `target_id`; the "cannot read back" sentence is dropped.
- row-validity M2 — MEDIUM — fixed as proposed: Card 4's keep-the-first-record row now fails the first `Call` through a `ReviewErrs` error before a second accepted `Call`.
- row-validity M3 — MEDIUM — fixed differently: Card 4 lists `halfProducerSpec`, `newReviewProducer` and `newFixProducer` under Edit as proposed, and, because `TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded` is a usage table over engines with no target, puts the snapshot rows in a new `TestBurlerReviewProducer_Call_RecordsRoundSnapshot` instead of forcing them into that table.
- coverage 1 — MEDIUM — fixed as proposed: `TestBouncer_Stalemate` gains rows whose ids are equal but whose fixer-written report misses a disposition or does not parse, each counting as changed.
- coverage 2 — MEDIUM — fixed as proposed: `TestBouncer_DecisionEscalation` gains a row where round N carries the still-open held finding and leaves out the resolved one, escalating with a detail naming only the carried id.
- coverage 3 — MEDIUM — fixed as proposed: `TestBuildRoundContext` gains `ParentReportPath` rows (ask with report, no ask, unreadable ask record) and `TestBurlerReviewProducer_Call_ProfileAndOpts` a parent-seat row asserting the path on the spawned and probed profile.
- coverage 4 — MEDIUM — fixed as proposed for the hard cap: `TestBouncer_Circling_NoDecisionAwaitsNamingBothVerbs` gains a hard-cap row whose notice names accept and not continue; the empty-slug branch is handled under row-validity L2.
- row-validity L1 — LOW — fixed as proposed: Card 7 lists `writeJudgedRound` under Uses and its rows take round 1's snapshot through the row's `TargetSnapshot.Take`.
- row-validity L2 — LOW — fixed as proposed: Card 9 drops the empty-slug clause (an empty slug already fails the notice fill, `internal/shedadapters/bouncer.go:1577-1582`) and asserts the notice on the slugged rows only.
- row-validity L3 — LOW — fixed as proposed: Card 4 drops the skipped-fix row.
- row-validity L4 — LOW — fixed as proposed: the skipped reports' parse is asserted once, in `TestCheckFixerReport`.
- row-validity L5 — LOW — fixed as proposed: Card 8 creates `internal/burlercli/validate_integration_test.go`, stated tier1, to hold `TestValidateReviewVerb_RoundChecks`.
- row-validity L6 — LOW — fixed as proposed: Card 5 drops `internal/burlerengine#FixerReport` from Uses.
- way-forward 2 / coverage 9 — LOW — fixed as proposed: Card 8's builder error falls back to the shape checks alone, printing ok with a `round_checks` field naming the error, and a row reaches it.
- way-forward 3 — LOW — fixed as proposed: Card 8 states the bound that the verb checks against the decision record as it stands, and the help text says so.
- way-forward 4 — LOW — fixed as proposed: Card 12's `LoadConfig` refusal ends with `reviewbudget.Route(dirs, ConfigSpec(), "review_stalemate_rounds")`, the layer-aware way forward.
- coverage 7 — LOW — fixed as proposed: `TestBouncer_Stalemate` gains an unreadable-ledger-in-window row escalating `budget` and a nil-`Snapshot` row never escalating.
- coverage 8 — LOW — fixed as proposed: `TestBuildRoundContext` gains rows for an absent and an unparseable round N-1 review, an absent snapshot record and a non-absence read error; `TestBurlerReviewProducer_Call_ProfileAndOpts` gains rows writing no changes file when round 1's record has no `target_id` and when `ChangedSince` answers false.
- coverage 10 — LOW — fixed as proposed: Cards 2 and 5 state their refusal rows assert the `way forward:` clause, and Card 2 adds a row running `FixerReportGateEntry`'s gate over an unparseable review.
- coverage 11 — LOW — fixed as proposed: Card 5 adds absolute-path and space-cut location rows reaching `locationPath`.
- coverage 12 — LOW — fixed as proposed: Card 6 adds rows asserting the block names round N-1's review and fixer-report paths and the changes file when `ChangesPath` is set.
- coverage 13 — LOW — fixed as proposed: Card 3 adds a missing-target-path row and a second-`Take` row.
- forced-edits 2 — MEDIUM — fixed as proposed: Card 5 moves `internal/burlerengine/doc.go` under Edit and rewrites the only-keys sentence to name `prior`.
- forced-edits 3 — MEDIUM — fixed as proposed: Card 8 lists `internal/burlerengine/doc.go` under Edit and rewrites its validate-review sentence.
- forced-edits 4 — LOW — fixed as proposed: Card 2 moves `internal/burlerengine/doc.go` under Edit and rewrites the skipped-fix sentence to state the Go-written frontmatter.
