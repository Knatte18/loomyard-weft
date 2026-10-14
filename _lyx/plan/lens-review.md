# Lens review

Every fork returned a report.

- forced-edits, BLOCKING, card 8: `InnerRunDeps.ArbiterStop` left nil in `envkit.FullEnv` fails `NilSeams` — fixed: card 8 adds `"InnerRun.ArbiterStop"` to `internal/testkit/envkit#nilLegal`, nil being the documented not-eligible default.
- forced-edits, LOW, card 3: `caps.go`'s file comment goes stale with `CapRoute` gone — fixed: card 3's `Config.Caps` bullet rewrites it.
- forced-edits, LOW, card 4: the darn build-time refusal row would lose its own row — fixed: card 4 keeps it and adds a load-time row.
- forced-edits, LOW, card 3: `docs/overview.md:511` never named `skip_merge_review` — fixed: the instruction now replaces "where 0 takes `review_max_bounces`" and adds `merge_max_bounces` to the list.
- row-reachability, MEDIUM, card 2: `ZeroRoundsApprovesAtOnce` over `minimalBouncerConfig` wires no approve or commit seam — fixed: the subtest sets `approve_seam: plan` and `commit_seam: plan`.
- row-reachability, MEDIUM, card 7: `TestHoldReading` named no escalation layout reachable from outside `shedadapters` — fixed: the card names `round-1-review.md` and `round-1-escalation.md` under Plan-Bouncer's run directory, the names `roundReviewPath` (`internal/shedadapters/burler.go:24`) and `escalationPath` (`internal/shedadapters/round.go:75`) give.
- row-reachability, MEDIUM, card 7: no `blocked` row in `TestHoldReading` — fixed: a blocked Plan-Write row reads eligible; the verify-halt refusal is `Check`'s, reached by `TestArbiterVerb_SpawnPath`'s existing Merge-In and Publish rows.
- row-reachability, LOW, card 7: loom.yaml seeding underspecified — fixed: the card seeds the template in the board dir and the anchor, as `hubLocation` does.
- row-reachability, LOW, card 3: `TestMigrateConfig`'s first row expected two lines from the full template — fixed: the row uses a hand-written fragment and lists its exact lines.
- row-reachability, LOW, and criteria-coverage, MEDIUM, card 8 (merged): no row pins that a start-report file does not end the hold — fixed: card 8 extends `stepStopReport_ReadsTheParkMarkersContentAndTime` with a drive report file and no park marker, asserting `StopReport` reports none.
- criteria-coverage, MEDIUM, card 9: D7 pinned only by a `mustNot` — fixed: a `must` of the fork-first wording in `## Error and interrupted stops` is added.
- criteria-coverage, MEDIUM, cards 3 and 4: no stated bound for switching any review off per run — fixed: each Intent states what a task can now skip, that no hub guard bounds it as the record accepts, and what still binds a perch above 0.
- criteria-coverage, LOW, card 4: no row checks a per-run darn budget reaches the Env — fixed: `TestWire_DarnRecipe` gains a local-override subtest asserting budget 2 and hard cap 3.
- criteria-coverage, LOW, card 6: the driver's one-line `EscalationPending` closure is untested — rejected: it is wiring glue whose two halves, `EscalationEligible` and `decideDrive`'s "awaiting with a parent notice" row (`internal/loomcli/driverules_test.go:39`), are each pinned.
- criteria-coverage, LOW, overview: the record's stale `skip_review` constraint and open risk — rejected: the overview already says the added entries override them, and the record is the parent's to edit.
- test-economy, LOW, card 6: eligibility asserted twice — fixed: `TestEscalationEligible` asserts only the driver fact; refusal texts stay with the verb rows.
- test-economy, LOW, cards 3 and 4: `TestPrepareRunSettings` repeats `TestCheckLocal`'s bounds — fixed for loom, now one acceptance and one refusal; darn keeps its three rows, each an acceptance criterion's own case.
- test-economy, LOW, card 1: `TestCheckResolved` repeats route coverage of consumer tests — rejected: card 1 lands before any consumer moves onto `Route`, so its own test is the only coverage of the package until card 3.
- test-economy, LOW, card 2: the zero-round settle counted twice — fixed: the loomrecipe test asserts Done with no spawn; seam counting stays in `TestBouncerEntry_SkipSeam`.
- test-economy, LOW, card 3: `TestMigrateConfig` could be configsync rows — rejected: `MigrateConfig` is loomengine's public surface, and configsync's rows drive reconcile, not the rewrite rules.
