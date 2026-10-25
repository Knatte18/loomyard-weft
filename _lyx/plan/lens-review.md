# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates are merged and listed once under the first lens that raised them.

## Findings

- coverage 1, MEDIUM — fixed differently: no new test; card 3's `buildSequenceFixture` lays out an open parent-review round and `TestSequence_FullRunBlocksAtMergeIn` asserts Discussion-Write passes to Discussion-Bouncer with the round left on disk, which reaches the in-flight criterion through the real gate list.
- coverage 2, MEDIUM — fixed as proposed: card 16's `recipeRow` maps a producer through `LoomRetiredRows` and `DarnRetiredRows`, and `TestPeersVerb` gains a peer parked on `Merge-In`.
- coverage 3, MEDIUM — fixed as proposed: card 12 rows 15 (unparseable review) and 16 (an escalation record of another cause) reach the two handled-false returns.
- coverage 4, MEDIUM — fixed as proposed: card 12 row 1 asserts the Awaiting Reason names `lyx loom decision add <slug>`, both circling verbs and `lyx loom resume`.
- coverage 5, LOW — fixed as proposed: card 12 row 10's next Call spawns a judge for the round.
- coverage 6, LOW — fixed as proposed: card 12 row 6 adds a `decision` finding whose id is an open MEDIUM ledger key, appearing once.
- coverage 7, LOW — fixed as proposed: card 12 row 1 gains a variant with no decision record, writing an empty snapshot.
- coverage 8, LOW — fixed as proposed: card 12 row 5 asserts not `BudgetExempt` below the budget and `BudgetExempt` at a spent one.
- coverage 9, LOW — fixed as proposed: card 13's fix-producer test gains a `DecisionsEscalated` row with no notifier.
- coverage 10, LOW — fixed as proposed: card 16's `TestReviewInputSeams` gains a both-present row, and the fixer test asserts the old record gone after both removal sites.
- coverage 11, LOW — disputed, premise false: `TestPrepareRunSettings`'s subtest "each migration line is logged at Warn naming the module, the key and the way forward" (`internal/configreg/runsettings_test.go:345`) already asserts the Warn every migrated line takes, and cards 6 and 15 assert `MigrateConfig` returns each line.
- coverage 12, LOW — fixed as proposed: card 3 corrects `internal/shuttleengine#GateEntry`'s `MayHold` sentence; `creelengine/doc.go`'s past-stage design note is left as the history the finding allows.
- row-validity 1, BLOCKING — fixed differently: card 4 deletes `TestResolvePersistentPreRun_SkipsReviewGroup` and creates `TestResolvePersistentPreRun_SkipsSlugGroups`, keeping its skip assertion over the remaining groups plus the no-`review`-child assertion, in place of the separate `TestCommand_HasNoReviewGroup`; editing the old test in place would leave a name that says it skips a group that no longer exists.
- row-validity 2, MEDIUM — fixed as proposed: card 13's Bouncer-entry rows overwrite the round-1 review with a `decision` finding and want Awaiting, with `layoutBouncerRound1Report` under `Uses`.
- row-validity 3, MEDIUM — fixed as proposed: card 12 row 3 starts round 2 unjudged and harvests through `judgeFakeShuttle`.
- row-validity 4, MEDIUM — fixed as proposed: card 13's fix-entry row sets both the notifier and `decisions_escalated`, wanting no line and `DecisionsRouted` true.
- row-validity 5, LOW — fixed as proposed: card 12 rows 4 and 13 run over `newReentry`, with `convergingShuttle` and `reentryOption` under `Uses`.
- row-validity 6, LOW — fixed as proposed: card 6's run-settings row states its expected `loom` document; the Warn is covered as coverage 11 says.
- guard-reach 1, MEDIUM (also way-forward 3 and forced-edits 1, BLOCKING) — fixed differently: card 16's `HandsBack` matches `Parent-Align` and the retired `Merge-In` alike and `TestHandsBack` keeps its `Merge-In` rows true beside new `Parent-Align` rows, not forced-edits' rewrite of those rows to `Parent-Align`, since a run halted on `Merge-In` at upgrade must still hand back on batten's direct read.
- guard-reach 2, LOW — fixed as proposed: card 12's `routeDecisions` returns handled false over a settled accept.
- guard-reach 3, LOW — fixed as proposed: card 12 names the reaching input (a cap lowered after a continue recorded below it), and row 12 lowers `HardCap` after the record.
- guard-reach 4, LOW — fixed as proposed: card 12's `routeDecisions` steps aside for a decision file of another cause, and row 17 reaches it.
- way-forward 1, MEDIUM — fixed as proposed: a held-refusal re-escalation's detail says an accept re-escalates while the refusal holds and names `continue`.
- way-forward 2, MEDIUM — fixed with own change, the finding proposing none: card 12's halt Reason opens with `DecisionHaltPrefix`, and card 14's `arbiterstop.Check` refuses a prefixed blocked stop with `HandBackWayForward` and batten's `HandBack` seam marks it, so only the operator's resume lifts it; rows in `TestCheck_RefusesLandingHalts`.
- way-forward 4, MEDIUM — fixed as proposed: card 16's decode refusal names the file it read, and a row refuses an undecodable old record naming the old path.
- way-forward 5, LOW — fixed as proposed: card 12's fragment takes Go-picked `decide_verbs`, the accept line alone at or past the hard cap.
- forced-edits 2, BLOCKING — fixed as proposed: card 16 keys `TestEntryNumbers` entry 8 `Parent-Align`.
- forced-edits 3, BLOCKING — fixed as proposed: card 6 lists `internal/webstercli#seedLoomConfigWithFriction`.
- forced-edits 4, MEDIUM — fixed as proposed: card 16 renames `envkit`'s `LandingDeps` gate key and `fixedGateLists` entry to `parent-align`.
- forced-edits 5, LOW — fixed differently: listing `TestMergeIn_Fixer` and `mergeInFixture` beside the card's `Rename` pair is refused by `redundant-file-target` in `lyx loom validate-plan` (the pair's old side is the file both resolve into), so the card keeps the file pairs as its targets and names both members and the new rows in its Intent.
- forced-edits 6, LOW — disputed, premise false: `grep -i "parent review\|parent-review" internal/loomcli/resume.go` matches nothing, and `internal/loomcli/wiring_commitstatus_test.go:161` names `parent-review` as the point segment of an archived alignment directory the status commit still stages, not text naming the step between Discussion-Write and Discussion-Review.
- forced-edits 7, LOW — fixed as proposed: card 16 makes `TestDarnWrite_Call` lack `Parent-Align`.
